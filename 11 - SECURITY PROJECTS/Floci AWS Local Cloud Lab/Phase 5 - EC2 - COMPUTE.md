# Phase 5: EC2 / COMPUTE

## 📖 Overview
This note documents the infrastructure-as-code steps required to provision the EC2 / COMPUTE for the Floci Local Cloud Lab.

## ⚙️ Key Variables
- `\$AMI_ID`
- `\$CSTATE`
- `\$EC2_CONTAINER`
- `\$EC2_KEY_NAME`
- `\$EC2_SSH_PORT`
- `\$EC2_USER_DATA`
- `\$EXISTING_KEY`
- `\$INSTANCE_ID`
- `\$INSTANCE_KEY`
- `\$KEY_PRESENT`
- `\$SG_ID`
- `\$SSHD_CFG`
- `\$SSHD_PID`
- `\$SSH_OK`
- `\$SSH_OUT`
- `\$SSH_PRIVATE_KEY`
- `\$SSH_PUBKEY_CONTENT`
- `\$SSH_PUBLIC_KEY`
- `\$SSH_READY`
- `\$STATE`
- `\$SUBNET_ID`

## 💻 Implementation Script
```bash
# PHASE 5 — EC2 / COMPUTE
# Provisions a real Amazon Linux 2 instance with:
#   - SSH key pair imported from $SSH_PUBLIC_KEY
#   - ec2-user created and authorized via UserData
#   - sshd hardened and running
#   - SSH port discovered and printed
###############################################################################

info "PHASE 5 — EC2 / COMPUTE"

# ---------------------------------------------------------------------------
# 5.0 — Validate SSH public key
# ---------------------------------------------------------------------------
if [[ ! -f "$SSH_PUBLIC_KEY" ]]; then
  die "SSH public key not found: $SSH_PUBLIC_KEY
  Set SSH_PUBLIC_KEY to the path of your lab SSH public key.
  Example: SSH_PUBLIC_KEY=~/.ssh/id_rsa.pub bash floci-lab.sh"
fi

SSH_PUBKEY_CONTENT=$(cat "$SSH_PUBLIC_KEY")
if [[ -z "$SSH_PUBKEY_CONTENT" ]]; then
  die "SSH public key is empty: $SSH_PUBLIC_KEY"
fi

success "SSH public key found: $SSH_PUBLIC_KEY"

# ---------------------------------------------------------------------------
# 5.1 — Check EC2 API availability
# ---------------------------------------------------------------------------
if ! aws_local ec2 describe-instances >/dev/null 2>&1; then
  warn "EC2 API unavailable — skipping EC2 phase"
else

# ---------------------------------------------------------------------------
# 5.2 — Discover AMI (Floci provides Amazon Linux 2 automatically)
# ---------------------------------------------------------------------------
AMI_ID=$(
  aws_local ec2 describe-images \
    --owners self amazon \
    --query 'Images[0].ImageId' \
    --output text 2>/dev/null
)

if [[ "$AMI_ID" == "None" || -z "$AMI_ID" ]]; then
  warn "No EC2 AMI available in this Floci runtime — skipping EC2 phase"
else

# ---------------------------------------------------------------------------
# 5.3 — Import SSH key pair (idempotent)
# Floci stores the public key and injects it into /root/.ssh on the container
# at launch time. The AWS CLI fileb:// prefix base64-encodes the file.
# ---------------------------------------------------------------------------
EXISTING_KEY=$(
  aws_local ec2 describe-key-pairs \
    --key-names "$EC2_KEY_NAME" \
    --query 'KeyPairs[0].KeyName' \
    --output text 2>/dev/null
)

if [[ "$EXISTING_KEY" == "$EC2_KEY_NAME" ]]; then
  success "EC2 key pair exists: $EC2_KEY_NAME"
else
  aws_local ec2 import-key-pair \
    --key-name "$EC2_KEY_NAME" \
    --public-key-material "fileb://${SSH_PUBLIC_KEY}" \
    >/dev/null 2>&1 \
    && success "Imported EC2 key pair: $EC2_KEY_NAME" \
    || warn "EC2 key pair import failed — SSH key injection may not work"
fi

# ---------------------------------------------------------------------------
# 5.4 — Check for existing instance
# If it has no key pair (broken legacy), terminate and recreate.
# Floci can only inject SSH keys at launch time — not retroactively.
# ---------------------------------------------------------------------------
INSTANCE_ID=$(
  aws_local ec2 describe-instances \
    --filters \
      "Name=tag:Name,Values=${LAB_PREFIX}-server" \
      "Name=instance-state-name,Values=running,stopped,pending" \
    --query 'Reservations[].Instances[0].InstanceId' \
    --output text 2>/dev/null
)

if [[ -n "$INSTANCE_ID" && "$INSTANCE_ID" != "None" ]]; then
  INSTANCE_KEY=$(
    aws_local ec2 describe-instances \
      --instance-ids "$INSTANCE_ID" \
      --query 'Reservations[0].Instances[0].KeyName' \
      --output text 2>/dev/null
  )
  if [[ -z "$INSTANCE_KEY" || "$INSTANCE_KEY" == "None" ]]; then
    warn "Existing instance $INSTANCE_ID has no key pair — terminating to recreate"
    warn "(Floci injects SSH keys at launch only — cannot be added retroactively)"
    aws_local ec2 terminate-instances \
      --instance-ids "$INSTANCE_ID" >/dev/null 2>&1 || true
    for _t in $(seq 1 20); do
      sleep 2
      STATE=$(
        aws_local ec2 describe-instances \
          --instance-ids "$INSTANCE_ID" \
          --query 'Reservations[0].Instances[0].State.Name' \
          --output text 2>/dev/null
      )
      [[ "$STATE" == "terminated" ]] && break
    done
    INSTANCE_ID=""
  else
    success "EC2 instance exists with key pair: $INSTANCE_ID (key: $INSTANCE_KEY)"
  fi
fi

# ---------------------------------------------------------------------------
# 5.5 — Launch instance with key pair + UserData
# UserData runs inside the container via docker exec AFTER Floci has:
#   1. injected the public key to /root/.ssh/authorized_keys
#   2. started sshd
# So UserData safely copies from /root/.ssh and reloads the running sshd.
# ---------------------------------------------------------------------------
if [[ -z "$INSTANCE_ID" || "$INSTANCE_ID" == "None" ]]; then
  if [[ -z "$SUBNET_ID" || -z "$SG_ID" ]]; then
    warn "No subnet or security group available — cannot launch EC2 instance"
  else

# UserData: best-effort hostname + convenience packages.
# ec2-user setup is handled reliably by the host-side docker exec block (5.5b)
# so we do NOT use set -e here — partial success is fine.
EC2_USER_DATA='#!/bin/bash
# Note: no set -e; individual failures are non-fatal.

# -- hostname ----------------------------------------------------------------
hostname cyber-lab-server 2>/dev/null || true
echo "cyber-lab-server" > /etc/hostname

# -- convenience packages (best-effort) --------------------------------------
yum install -y sudo curl wget net-tools iproute procps >/dev/null 2>&1 || true

echo "[UserData] hostname and packages complete"

# -- ec2-user (belt-and-suspenders fallback in case docker exec step missed) --
id -u ec2-user >/dev/null 2>&1 || useradd -m -s /bin/bash ec2-user || true
mkdir -p /home/ec2-user/.ssh || true
if [[ -f /root/.ssh/authorized_keys ]]; then
  cp /root/.ssh/authorized_keys /home/ec2-user/.ssh/authorized_keys 2>/dev/null || true
  chown -R ec2-user:ec2-user /home/ec2-user/.ssh 2>/dev/null || true
  chmod 700 /home/ec2-user/.ssh 2>/dev/null || true
  chmod 600 /home/ec2-user/.ssh/authorized_keys 2>/dev/null || true
fi

echo "[UserData] done"'

    INSTANCE_ID=$(
      aws_local ec2 run-instances \
        --image-id "$AMI_ID" \
        --instance-type t3.micro \
        --key-name "$EC2_KEY_NAME" \
        --subnet-id "$SUBNET_ID" \
        --security-group-ids "$SG_ID" \
        --user-data "$EC2_USER_DATA" \
        --tag-specifications \
          "ResourceType=instance,Tags=[{Key=Name,Value=${LAB_PREFIX}-server}]" \
        --query 'Instances[0].InstanceId' \
        --output text 2>/dev/null
    )

    if [[ -n "$INSTANCE_ID" && "$INSTANCE_ID" != "None" ]]; then
      success "Created EC2 instance: $INSTANCE_ID"
    else
      warn "EC2 instance creation failed"
      INSTANCE_ID=""
    fi
  fi  # subnet/sg guard
fi    # needs new instance

# ---------------------------------------------------------------------------
# 5.5b — Reliable ec2-user setup via docker exec
# UserData runs asynchronously and can fail silently on minimal images.
# This block runs from the host immediately after the container is confirmed
# running, guaranteeing ec2-user + authorized_keys are in place before the
# SSH validation in 5.7. Runs whether the instance is new or pre-existing.
# ---------------------------------------------------------------------------
if [[ -n "$INSTANCE_ID" && "$INSTANCE_ID" != "None" ]]; then
  EC2_CONTAINER="floci-ec2-${INSTANCE_ID}"
  # Wait for container to be running (max 30 s)
  for _c in $(seq 1 15); do
    CSTATE=$(docker inspect "$EC2_CONTAINER" --format '{{.State.Status}}' 2>/dev/null)
    [[ "$CSTATE" == "running" ]] && break
    sleep 2
  done
  if [[ "$(docker inspect "$EC2_CONTAINER" --format '{{.State.Status}}' 2>/dev/null)" == "running" ]]; then
    # Wait up to 20 s for Floci to inject the public key into /root/.ssh
    for _k in $(seq 1 10); do
      KEY_PRESENT=$(docker exec "$EC2_CONTAINER" /bin/sh -c \
        'test -s /root/.ssh/authorized_keys && echo yes || echo no' 2>/dev/null)
      [[ "$KEY_PRESENT" == "yes" ]] && break
      sleep 2
    done
    # Set up ec2-user, authorized_keys, sudo, and sshd hardening
    docker exec "$EC2_CONTAINER" /bin/bash -c '
      # ec2-user
      id -u ec2-user >/dev/null 2>&1 || useradd -m -s /bin/bash ec2-user

      # authorized_keys — copy from /root/.ssh which Floci has already populated
      mkdir -p /home/ec2-user/.ssh
      cp /root/.ssh/authorized_keys /home/ec2-user/.ssh/authorized_keys 2>/dev/null || touch /home/ec2-user/.ssh/authorized_keys
      chown -R ec2-user:ec2-user /home/ec2-user/.ssh
      chmod 700 /home/ec2-user/.ssh
      chmod 600 /home/ec2-user/.ssh/authorized_keys

      # sudo access (install sudo if not present)
      command -v sudo >/dev/null 2>&1 || yum install -y sudo >/dev/null 2>&1 || true
      mkdir -p /etc/sudoers.d
      echo "ec2-user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ec2-user
      chmod 440 /etc/sudoers.d/ec2-user

      # sshd hardening
      SSHD_CFG=/etc/ssh/sshd_config
      grep -q "^PermitRootLogin no" "$SSHD_CFG" || \
        sed -i "s/^.*PermitRootLogin.*/PermitRootLogin no/" "$SSHD_CFG"
      grep -q "^PasswordAuthentication no" "$SSHD_CFG" || \
        sed -i "s/^.*PasswordAuthentication.*/PasswordAuthentication no/" "$SSHD_CFG"
      grep -q "^PubkeyAuthentication yes" "$SSHD_CFG" || \
        { grep -q "PubkeyAuthentication" "$SSHD_CFG" && \
          sed -i "s/^.*PubkeyAuthentication.*/PubkeyAuthentication yes/" "$SSHD_CFG"; } || \
          echo "PubkeyAuthentication yes" >> "$SSHD_CFG"

      # reload sshd to apply config (find PID from /var/run/sshd.pid or /proc)
      SSHD_PID=$(cat /var/run/sshd.pid 2>/dev/null || \
        grep -rl sshd /proc/[0-9]*/comm 2>/dev/null | head -1 | grep -o "[0-9]*" | head -1 || true)
      [ -n "$SSHD_PID" ] && kill -HUP "$SSHD_PID" 2>/dev/null || true
    ' 2>&1 | sed "s/^/    [exec] /" \
      && success "ec2-user provisioned inside $EC2_CONTAINER" \
      || warn "ec2-user setup had errors — SSH may still work"
  else
    warn "Container $EC2_CONTAINER not in running state — skipping ec2-user setup"
  fi
fi

# ---------------------------------------------------------------------------
# 5.6 — Discover SSH host port allocated by Floci
# Floci binds a unique host port per instance (default range starts at 2200).
# ---------------------------------------------------------------------------
if [[ -n "$INSTANCE_ID" && "$INSTANCE_ID" != "None" ]]; then
  for _w in $(seq 1 20); do
    EC2_CONTAINER="floci-ec2-${INSTANCE_ID}"
    EC2_SSH_PORT=$(
      docker inspect "$EC2_CONTAINER" \
        --format '{{range $p,$b := .NetworkSettings.Ports}}{{if eq $p "22/tcp"}}{{(index $b 0).HostPort}}{{end}}{{end}}' \
        2>/dev/null
    )
    [[ -n "$EC2_SSH_PORT" ]] && break
    sleep 2
  done

  if [[ -n "$EC2_SSH_PORT" ]]; then
    success "EC2 SSH host port: $EC2_SSH_PORT"
  else
    warn "Could not discover SSH host port for $INSTANCE_ID — container may still be starting"
  fi

# ---------------------------------------------------------------------------
# 5.7 — Wait for SSH readiness, then validate
# Floci runs startSshd asynchronously; UserData runs after. Allow 90 s total.
# ---------------------------------------------------------------------------
  if [[ -n "$EC2_SSH_PORT" ]]; then
    info "Waiting for SSH on 127.0.0.1:${EC2_SSH_PORT} (up to 90 s) ..."
    SSH_READY=0
    for _s in $(seq 1 45); do
      if nc -z -w2 127.0.0.1 "$EC2_SSH_PORT" >/dev/null 2>&1; then
        SSH_READY=1
        break
      fi
      sleep 2
    done

    if [[ $SSH_READY -eq 1 ]]; then
      success "SSH port is reachable: 127.0.0.1:${EC2_SSH_PORT}"
      SSH_PRIVATE_KEY="${SSH_PUBLIC_KEY%.pub}"
      if [[ -f "$SSH_PRIVATE_KEY" ]]; then
        SSH_OUT=$(
          ssh \
            -o StrictHostKeyChecking=no \
            -o UserKnownHostsFile=/dev/null \
            -o ConnectTimeout=10 \
            -o BatchMode=yes \
            -i "$SSH_PRIVATE_KEY" \
            -p "$EC2_SSH_PORT" \
            ec2-user@127.0.0.1 \
            'printf "whoami=%s\nhostname=%s\nuname=%s\n" "$(whoami)" "$(hostname)" "$(uname -r)"; ip addr show | awk "/inet .* scope global/{print \"ip=\"\$2}"' \
            2>/dev/null
        ) && SSH_OK=0 || SSH_OK=$?
        if [[ $SSH_OK -eq 0 ]]; then
          success "SSH validation passed:"
          echo "$SSH_OUT" | sed 's/^/    /'
        else
          warn "SSH TCP is open but ec2-user login failed"
          warn "Check: docker exec floci-ec2-${INSTANCE_ID} ls -la /home/ec2-user/.ssh/"
          warn "Retry: ssh -i $SSH_PRIVATE_KEY -p $EC2_SSH_PORT ec2-user@127.0.0.1"
        fi
      else
        warn "Private key not found at $SSH_PRIVATE_KEY — skipping live SSH test"
      fi
    else
      warn "SSH port did not become reachable within 90 s"
      warn "Check: docker logs floci-ec2-${INSTANCE_ID}"
    fi

# ---------------------------------------------------------------------------
# 5.8 — Print connection info
# ---------------------------------------------------------------------------
    EC2_PRIVATE_IP=$(
      aws_local ec2 describe-instances \
        --instance-ids "$INSTANCE_ID" \
        --query 'Reservations[0].Instances[0].PrivateIpAddress' \
        --output text 2>/dev/null
    )
    echo
    echo "=================================================================="
    echo " EC2 SSH ACCESS"
    echo "=================================================================="
    echo "  Instance:   ${LAB_PREFIX}-server ($INSTANCE_ID)"
    echo "  Private IP: ${EC2_PRIVATE_IP:-unknown}"
    echo "  SSH port:   ${EC2_SSH_PORT}  (host-forwarded from container :22)"
    echo "  Key pair:   $EC2_KEY_NAME"
    echo
    echo "  Connect:"
    echo "    ssh -i ${SSH_PUBLIC_KEY%.pub} -p ${EC2_SSH_PORT} ec2-user@127.0.0.1"
    echo
    echo "  Inside instance:"
    echo "    whoami        -> ec2-user"
    echo "    hostname      -> cyber-lab-server"
    echo "    sudo -n true  -> (no output = success)"
    echo "    ip addr"
    echo
    echo "  SSH persists across: docker restart, StopInstances/StartInstances"
    echo "  Filesystem is ephemeral — recreating terminates container data"
    echo
    echo "  Troubleshoot:"
    echo "    docker ps | grep ${INSTANCE_ID}"
    echo "    docker exec floci-ec2-${INSTANCE_ID} cat /var/run/sshd.pid"
    echo "=================================================================="
    echo
  fi  # EC2_SSH_PORT guard
fi    # INSTANCE_ID guard

fi  # AMI guard
fi  # EC2 API guard



###############################################################################
```

*Note: This script is designed to be idempotent. It will check if resources exist before creating them.*
