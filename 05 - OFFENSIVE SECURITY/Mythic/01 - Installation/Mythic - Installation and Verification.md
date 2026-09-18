---
title: Mythic - Installation and Verification
category: Installation
environment: GOAD
execution_context: "[KALI]"
privileges: root
tags:
  - Mythic
  - Installation
  - Setup
  - Kali
date_created: 2026-08-23
status: CORRECTED
---

# Mythic — Installation & Verification

> [!WARNING]
> This document corrects errors found in the existing vault notes.
> The GOAD Assault Playbook used the wrong GitHub repository (`IT-007/Mythic`).
> See [[GOAD - Error Audit and Contradiction Report]] ERROR 10.

> [!NOTE]
> All commands in this note run on **Kali only**. None of these commands execute on GOAD VMs.

---

## Prerequisites

- [ ] Kali Linux (or Ubuntu/Debian) with internet access
- [ ] Docker and Docker Compose installed (or use the provided install script)
- [ ] At least 4 GB RAM for Mythic containers
- [ ] Git installed
- [ ] Kali has connectivity to GOAD network (192.168.56.0/24) — verify [[GOAD - Environment Setup and Validation]]

---

## Step 1 — Clone the Correct Repository

```
Execution Context: [KALI]
Shell: Bash
Privileges: Normal user
```

> [!CAUTION]
> The existing GOAD Assault Playbook used `https://github.com/IT-007/Mythic.git` — this repository does NOT exist.
> The correct repository is below.

```bash
# Clone the official Mythic repository
git clone https://github.com/its-a-feature/Mythic.git
cd Mythic
```

**Expected result:** Repository cloned successfully. `ls` shows `mythic-cli`, `Makefile`, etc.

**If it fails:** Check internet connectivity on Kali. `ping github.com`

---

## Step 2 — Install Docker (First Time Only)

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
# Use Mythic's provided script (Ubuntu/Debian/Kali)
sudo ./install_docker_ubuntu.sh

# Verify Docker is running
sudo systemctl status docker
```

**Expected result:** `docker` service is active.

> [!NOTE]
> If Docker is already installed, skip this step. Check with `docker --version`.

---

## Step 3 — Start Mythic

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
cd Mythic
sudo ./mythic-cli start
```

**Expected result:** Docker containers start. Output similar to:
```
[+] Starting Mythic services...
[+] mythic_server       ... done
[+] mythic_postgres     ... done
[+] mythic_rabbitmq     ... done
[+] mythic_nginx        ... done
```

**Verify containers are healthy:**
```bash
sudo docker ps
```

All containers should show `Up` and `healthy` status.

---

## Step 4 — Install Apollo Agent

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

> [!CAUTION]
> The GOAD Assault Playbook installed Apollo twice from two different repos. Use only the command below.

```bash
# Install Apollo (correct repository)
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo

# Install Athena (optional, multi-platform)
sudo ./mythic-cli install github https://github.com/MythicAgents/Athena

# Install HTTP C2 profile
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http

# Restart to apply new agents
sudo ./mythic-cli restart
```

**Expected result:** Apollo and HTTP profile appear in Mythic UI after restart.

---

## Step 5 — Retrieve Login Credentials

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
# Get auto-generated admin password
sudo ./mythic-cli config get MYTHIC_ADMIN_PASSWORD
sudo ./mythic-cli config get MYTHIC_ADMIN_USER
```

Record these credentials securely.

---

## Step 6 — Access the Mythic UI

```
Execution Context: [KALI]
Shell: Browser
Privileges: N/A
```

```
Execution Context: [KALI]
URL: https://127.0.0.1:7443
```

> [!NOTE]
> The UI uses a self-signed certificate. Your browser will show a certificate warning — this is expected.
> Click "Advanced" → "Accept the risk" (Firefox) or equivalent.

Log in with the credentials from Step 5.

**Expected result:** Mythic dashboard loads. Menu shows Callbacks, Payloads, Files, etc.

---

## Step 7 — Upload Supporting Binaries to Mythic File Host

```
Execution Context: [KALI]
Shell: Bash
Privileges: Normal user
```

First, download the required binaries to Kali:

```bash
mkdir -p ~/payloads/goad
cd ~/payloads/goad

# GodPotato (.NET 4)
wget 'https://github.com/BeichenDream/GodPotato/releases/latest/download/GodPotato-NET4.exe' -O GodPotato.exe

# Mimikatz
wget 'https://github.com/gentilkiwi/mimikatz/releases/latest/download/mimikatz_trunk.zip'
unzip -o mimikatz_trunk.zip
cp x64/mimikatz.exe .

# SharpHound (BloodHound collector)
wget 'https://github.com/BloodHoundAD/BloodHound/raw/master/Collectors/SharpHound.exe'

# Rubeus (Kerberos toolkit)
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/Rubeus.exe'

# Seatbelt (host recon)
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/Seatbelt.exe'

# SharpUp (privilege escalation checks)
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/SharpUp.exe'
```

Then upload each to Mythic:
- In Mythic UI → **Payloads** → **File Host**
- Upload each `.exe` file

**Why this matters:** `execute-assembly` loads binaries from the Mythic file host — not from Kali's local disk.

---

## Verification Checklist

- [ ] `git clone https://github.com/its-a-feature/Mythic.git` succeeded
- [ ] Docker containers are running (`sudo docker ps` — all `healthy`)
- [ ] Mythic UI is accessible at `https://127.0.0.1:7443`
- [ ] Apollo agent appears in Mythic UI (Payloads → Create → Payload Type)
- [ ] HTTP C2 profile appears in Mythic UI
- [ ] All required binaries downloaded to `~/payloads/goad/`
- [ ] Binaries uploaded to Mythic File Host

---

## Common Mistakes

- Cloning the wrong repository (IT-007/Mythic does not exist)
- Forgetting to restart Mythic after installing new agents
- Installing Apollo twice (from both its-a-feature and MythicAgents)
- Not uploading binaries to Mythic File Host before generating payloads
- Accessing UI via HTTP instead of HTTPS

---

## Troubleshooting

### Containers fail to start

```bash
sudo docker ps -a
sudo docker logs mythic_server
sudo ./mythic-cli logs mythic_server
```

### Port 7443 is not accessible

```bash
# Check if mythic_nginx is running
sudo docker ps | grep nginx

# Check port binding
sudo ss -tlnp | grep 7443
```

### Agent not appearing after install

```bash
# Check agent installed correctly
sudo ./mythic-cli install list

# Force restart
sudo ./mythic-cli stop
sudo ./mythic-cli start
```

---

## Related Notes

- [[Mythic - Architecture Overview]]
- [[Mythic - Payload Generation]]
- [[GOAD - Environment Setup and Validation]]
- [[GOAD - Error Audit and Contradiction Report]]
