# Floci EC2 SSH Provisioning — Implementation Plan

## Background

Full inspection of the running Floci environment, the EC2 container, and the Floci source code
(`Ec2ContainerManager.java`) has been completed. Here is what was found.

---

## Findings

### What Floci's EC2 system actually does

Floci runs each EC2 instance as a real Docker container using the genuine Amazon Linux 2 image
(`public.ecr.aws/amazonlinux/amazonlinux:2`). After the container starts, Floci's Java daemon
asynchronously:

1. **Injects the SSH public key** (if a key pair was attached at `run-instances` time) into
   `/root/.ssh/authorized_keys` inside the container — **not** into `ec2-user`.
2. **Installs and starts sshd** by running `yum install -y openssh-server openssh-clients` (if
   not already present) and then executing `/usr/sbin/sshd` (daemonizes itself — no systemd needed).
3. **Executes UserData** shell scripts via `docker exec` (tar-uploads the script, runs it directly
   honoring its shebang). Works without systemd/cloud-init.
4. **Allocates SSH host ports** dynamically from a configurable range (default starts at 2200) —
   multiple instances each get a unique port automatically. Floci manages this; the script does
   not need to do it manually.
5. **Starts sshd unconditionally** — even if no key pair is given (matching real AWS). So the
   absence of `--key-name` in `run-instances` is why SSH worked at the port level in a previous
   run but produced `Connection reset` (no authorized key, just TLS/auth rejection).

### Why SSH is currently broken

The current `run-instances` call in `floci-lab.sh` does **not** pass `--key-name`, so Floci:
- Starts sshd ✅
- Does NOT inject any public key ❌
- The container has no `ec2-user` — only `root` ❌

Result: sshd is running, port 2200 is forwarded, but there is no authorized key so every
connection is rejected with `Connection reset`.

### Why there is no `ec2-user`

Floci does not create `ec2-user` — it injects keys into `/root/.ssh/authorized_keys`. The
real Amazon Linux 2 AMI creates `ec2-user` at first boot via cloud-init, but Floci containers
do not run cloud-init. `ec2-user` must be created via UserData.

### Key-pair flow in Floci

```
aws ec2 import-key-pair --key-name lab-key --public-key-material <base64(pubkey)>
                        ↓
        Floci stores key name → public key text
                        ↓
aws ec2 run-instances --key-name lab-key ...
                        ↓
        Floci looks up the public key material, injects into /root/.ssh/authorized_keys
        (and later ec2-user via UserData if we set that up)
```

### Port allocation

Floci allocates SSH host ports automatically from its configured range (default 2200+).
The first instance gets 2200, the second gets 2201, etc. The script does NOT need to
manage port numbers — it just needs to query the result after launch.

### Container CMD

The container runs `tail -f /dev/null` as PID 1. sshd daemonizes itself inside the container
and is NOT supervised. This means:
- sshd survives `docker restart` only if it is started in the UserData script or re-triggered
  by Floci on start. Floci does re-run `startSshd` on `StartInstances` API calls.
- The container filesystem is ephemeral (no volumes by default) — anything written inside
  disappears on container recreation.

### The old broken instance

`i-e8092fd6241642f50` (from the previous successful run) has no key pair and no `ec2-user`.
It must be terminated and replaced, or the new instance provisioning can simply detect its
state and terminate/recreate it when no key name is set.

---

## What Will Change

Only **Phase 5 (EC2)** of `floci-lab.sh` is modified. All other phases are untouched.

### New additions to the CONFIGURATION section

```bash
# SSH public key for EC2 instance access
SSH_PUBLIC_KEY="${SSH_PUBLIC_KEY:-$HOME/.ssh/id_ed25519.pub}"
EC2_KEY_NAME="${EC2_KEY_NAME:-${LAB_PREFIX}-key}"
SSH_FORWARDED_PORT=""   # filled in after instance launch
```

### Phase 5 changes — step by step

#### Step 1: Validate SSH public key

Before any EC2 work, check that `$SSH_PUBLIC_KEY` exists and is readable. Fail with a
clear error if it doesn't (never silently proceed with no key).

#### Step 2: Import key pair (idempotent)

```bash
aws ec2 import-key-pair \
  --key-name "$EC2_KEY_NAME" \
  --public-key-material fileb://"$SSH_PUBLIC_KEY"
```

If the key pair already exists with that name, skip (detect via `describe-key-pairs`).

> **Note:** Floci's `import-key-pair` takes base64-encoded material. The AWS CLI
> `fileb://` form handles this encoding automatically.

#### Step 3: Check for existing instance

Look for an instance tagged `Name=${LAB_PREFIX}-server`.

If the existing instance has **no key name** (the old broken one), terminate it and
recreate so Floci injects the key properly. (Key injection only happens at launch time.)

#### Step 4: Launch new instance with `--key-name` and UserData

The UserData script:

```bash
#!/bin/bash
# Create ec2-user if absent
id -u ec2-user &>/dev/null || useradd -m -s /bin/bash ec2-user
# sudo access
echo 'ec2-user ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ec2-user
chmod 440 /etc/sudoers.d/ec2-user
# Copy root's authorized_keys to ec2-user (Floci injects to /root)
mkdir -p /home/ec2-user/.ssh
cp /root/.ssh/authorized_keys /home/ec2-user/.ssh/authorized_keys
chown -R ec2-user:ec2-user /home/ec2-user/.ssh
chmod 700 /home/ec2-user/.ssh
chmod 600 /home/ec2-user/.ssh/authorized_keys
# Harden sshd
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#\?PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
# Set hostname
hostnamectl set-hostname cyber-lab-server 2>/dev/null || \
  echo "cyber-lab-server" > /etc/hostname
# Reload sshd to pick up config changes (sshd is already running at this point)
kill -HUP "$(cat /var/run/sshd.pid 2>/dev/null)" 2>/dev/null || true
```

Key points:
- UserData runs **after** Floci injects the public key to `/root/.ssh/` and starts sshd.
  So copying from `/root/.ssh/authorized_keys` to `ec2-user` is safe at UserData time.
- No systemd needed — sshd is already daemonized by Floci's `startSshd` before UserData runs.
- `hostnamectl` is tried first; falls back to `/etc/hostname` write.

#### Step 5: Discover the SSH host port

After launch, query the Floci-allocated port:

```bash
SSH_FORWARDED_PORT=$(docker inspect "floci-ec2-${INSTANCE_ID}" \
  --format '{{range $p, $b := .NetworkSettings.Ports}}{{if eq $p "22/tcp"}}{{(index $b 0).HostPort}}{{end}}{{end}}' \
  2>/dev/null)
```

#### Step 6: Wait for SSH readiness and print connection info

Wait up to 60s for TCP port to accept connections, then print:

```
==================================================================
 EC2 SSH ACCESS
==================================================================
Instance:   cyber-lab-server (i-xxxxxxxxxxxxxxxxx)
Private IP: 10.50.1.10
SSH port:   2200 (host-forwarded)

  ssh -i ~/.ssh/id_ed25519 -p 2200 ec2-user@127.0.0.1

==================================================================
```

#### Step 7: Validation block (automated)

The script runs the following checks automatically:

```bash
nc -vz 127.0.0.1 "$SSH_FORWARDED_PORT"   # TCP reachability
ssh -o StrictHostKeyChecking=no \
    -i "${SSH_PUBLIC_KEY%.pub}" \
    -p "$SSH_FORWARDED_PORT" \
    ec2-user@127.0.0.1 \
    "whoami; hostname; ip addr show; uname -a"
```

---

## User Review Required

> [!IMPORTANT]
> **The existing `i-e8092fd6241642f50` instance has no key pair.** The new code will
> detect this and **terminate it before recreating it**. This is necessary because
> Floci only injects SSH keys at launch time — there is no API to add a key to a
> running instance. All data inside that container will be lost (it has no persistent
> volumes anyway — the filesystem is ephemeral).

> [!IMPORTANT]
> **The SSH login user is `ec2-user`, but Floci injects keys to `/root`.**
> The UserData script bridges this by copying `/root/.ssh/authorized_keys` to
> `/home/ec2-user/.ssh/authorized_keys`. This is safe because Floci guarantees key
> injection completes before UserData runs (per the source code order in
> `Ec2ContainerManager.java` lines 509–523).

> [!WARNING]
> **No persistent storage.** Floci EC2 containers have no Docker volumes by default.
> Any files created inside the instance are lost when the container is recreated
> (e.g., on `TerminateInstances` + new `RunInstances`). This is a Floci limitation
> that matches the ephemeral nature of EC2 instances — use EBS volumes in real AWS.
> For the lab, the UserData re-runs on every new instance, so setup is always
> repeated correctly.

> [!NOTE]
> **SSH persists across `docker restart`** as long as the container is not
> terminated/recreated via the Floci API. Floci's `StartInstances` re-runs `startSshd`
> automatically. UserData does NOT re-run on start (only at initial launch) — this is
> correct behavior matching real EC2.

---

## Open Questions

None — all required information was gathered from the running containers and Floci source.

---

## Proposed Changes

### [MODIFY] [floci-lab.sh](file:///home/camara/floci-lab.sh)

#### Configuration section (lines ~63–116)
Add `SSH_PUBLIC_KEY` and `EC2_KEY_NAME` variables.

#### Phase 5 — EC2 / COMPUTE (lines ~1037–1143)
Complete rewrite of this section only. All other phases unchanged.

---

## Verification Plan

### Automated (run by the script itself)
- `nc -vz 127.0.0.1 <port>` — TCP reachability
- `ssh ... whoami; hostname; ip addr; uname -a` — live SSH session validation

### Manual verification commands

```bash
# Check container is running
docker ps | grep floci-ec2-i-

# Check sshd is running inside
docker exec floci-ec2-<id> /bin/sh -c 'cat /var/run/sshd.pid && ps -p $(cat /var/run/sshd.pid)'

# SSH connection
ssh -i ~/.ssh/id_ed25519 -p 2200 ec2-user@127.0.0.1

# Restart container and re-test SSH
docker restart floci-ec2-<id>
sleep 5
ssh -i ~/.ssh/id_ed25519 -p 2200 ec2-user@127.0.0.1 whoami
```

### Port collision test
Run script twice. Second invocation detects existing instance, skips recreation.
To test second instance: add a second server block with a different `LAB_PREFIX` value.

---

## Floci Limitations

| Feature | Status |
|---|---|
| UserData execution | ✅ Fully supported (docker exec, shell scripts) |
| SSH key injection | ✅ Supported — to `/root/.ssh` only |
| `ec2-user` | ⚠️ Must be created via UserData (not native Floci) |
| Port forwarding | ✅ Automatic, unique per instance |
| Persistent storage | ❌ No EBS/volume support — ephemeral only |
| systemd inside container | ❌ Not available — container runs `tail -f /dev/null` as PID 1 |
| sshd auto-restart | ✅ On `StartInstances` API call; ⚠️ NOT on raw `docker restart` (daemon restart) |
| Stop/start lifecycle | ✅ `StopInstances` stops container, `StartInstances` restarts + re-runs `startSshd` |
| cloud-init | ❌ Not available — Floci runs UserData directly via docker exec |
| Internet access | ✅ Container has internet via bridge network |
| Multiple instances | ✅ Each gets unique SSH port automatically |
