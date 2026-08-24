---
title: GOAD - Network Configuration and /etc/hosts Procedure
category: Network
environment: GOAD
execution_context: "[KALI]"
tags:
  - GOAD
  - Network
  - DNS
  - hosts
  - Kali
date_created: 2026-08-23
status: VERIFY-BEFORE-USE
---

# GOAD — Network Configuration & /etc/hosts Procedure

> [!WARNING]
> This procedure modifies `/etc/hosts` on your **Kali machine only**.
> It does not affect GOAD VMs or any external system.

> [!IMPORTANT]
> Verify the actual GOAD IP addresses before editing /etc/hosts.
> See [[GOAD - IP Hostname Matrix]] and [[GOAD - Error Audit and Contradiction Report]].

---

## Prerequisites

- [ ] GOAD environment is running
- [ ] Kali has a network interface on the 192.168.56.0/24 network
- [ ] You have confirmed the actual IP addresses with `nmap -sn 192.168.56.0/24`
- [ ] You know the interface name for the host-only adapter

---

## Step 1 — Inspect Current Configuration

```
Execution Context: [KALI]
Host: Kali attacker machine
Shell: Bash
Privileges: User
```

```bash
# View current /etc/hosts
cat /etc/hosts

# Check Kali's current IP on the GOAD network
ip addr show
# Identify which interface has a 192.168.56.x address

# Verify current DNS resolution
getent hosts kingslanding.sevenkingdoms.local
getent hosts winterfell.north.sevenkingdoms.local
getent hosts meereen.essos.local
```

**Expected result:** Either no entries exist for GOAD hostnames, or existing entries that need verification.

---

## Step 2 — Back Up the File

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
# Create a timestamped backup
sudo cp /etc/hosts /etc/hosts.backup.$(date +%Y%m%d_%H%M%S)

# Verify backup exists
ls -la /etc/hosts.backup.*
```

> [!TIP]
> Always back up /etc/hosts before modifying it. A malformed hosts file can break all DNS resolution on Kali.

---

## Step 3 — Edit the File

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
sudo nano /etc/hosts
```

Add the following block **after** the existing localhost entries.

> [!IMPORTANT]
> Replace the IPs below only AFTER you have verified them from your running GOAD deployment.
> Do NOT blindly copy these IPs — they are from the documented `/etc/hosts` which has NOT been verified against the live lab.

```text
# GOAD — Game of Active Directory
# IPs verified: VERIFY DATE
# Source: /etc/hosts reference + lab verification

192.168.56.10   sevenkingdoms.local kingslanding.sevenkingdoms.local kingslanding
192.168.56.11   winterfell.north.sevenkingdoms.local north.sevenkingdoms.local winterfell
192.168.56.12   essos.local meereen.essos.local meereen
192.168.56.22   castelblack.north.sevenkingdoms.local castelblack
192.168.56.23   braavos.essos.local braavos
```

**Save:** `Ctrl+O`, then `Enter`. **Exit:** `Ctrl+X`.

---

## Step 4 — Validate Syntax

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Verify the GOAD section was added correctly
grep -A 10 "GOAD" /etc/hosts

# Check for syntax errors (no tabs, correct format)
cat -A /etc/hosts | grep "56\."
# Should show: IP<space>hostname — no ^I (tab) characters
```

**Expected result:** Clean entries with spaces, no tab characters, correct IP format.

---

## Step 5 — Test Resolution

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

Test each hostname individually:

```bash
getent hosts kingslanding.sevenkingdoms.local
getent hosts kingslanding
getent hosts winterfell.north.sevenkingdoms.local
getent hosts north.sevenkingdoms.local
getent hosts winterfell
getent hosts meereen.essos.local
getent hosts essos.local
getent hosts meereen
getent hosts castelblack.north.sevenkingdoms.local
getent hosts castelblack
getent hosts braavos.essos.local
getent hosts braavos
```

**Expected result:** Each command returns the IP and hostname pair.

**If a hostname does not resolve:** The entry in /etc/hosts is either missing or has a typo. Re-check Step 3.

---

## Step 6 — Test Network Connectivity

> [!IMPORTANT]
> Successful hostname resolution does NOT prove the target is reachable.
> DNS resolution and network connectivity are separate.

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Test ICMP reachability to each host
ping -c 2 192.168.56.10
ping -c 2 192.168.56.11
ping -c 2 192.168.56.12
ping -c 2 192.168.56.22
ping -c 2 192.168.56.23

# Test with hostnames (after /etc/hosts is configured)
ping -c 2 kingslanding
ping -c 2 winterfell
ping -c 2 meereen
ping -c 2 castelblack
ping -c 2 braavos
```

**Expected result:** TTL values from Windows hosts are typically 128. Linux hosts show TTL 64.

---

## Step 7 — Troubleshooting

### Hostname does not resolve

```bash
# Check the /etc/hosts entry exists
grep kingslanding /etc/hosts

# Check /etc/nsswitch.conf — 'files' must come before 'dns'
grep hosts /etc/nsswitch.conf
# Expected: hosts: files dns
```

If `files` is not before `dns`, the hosts file will be bypassed:
```bash
sudo nano /etc/nsswitch.conf
# Change: hosts: dns files
# To:     hosts: files dns
```

### Hostname resolves but ping fails

```bash
# Confirm the IP is correct
getent hosts kingslanding
# Then ping the IP directly
ping -c 2 192.168.56.10

# Check if GOAD VM is running
# (Run from the GOAD directory on the host machine — not Kali)
# vagrant status
```

Possible causes:
- GOAD VM is not running
- VirtualBox host-only adapter is not enabled
- Windows Firewall on the GOAD VM is blocking ICMP

### IP responds but expected service is unavailable

```bash
# Check if the specific port is open
nmap -p 445,389,88,3389 192.168.56.10

# Check SMB specifically
smbclient -L //192.168.56.10 -N
```

Possible causes:
- Windows service is stopped on the GOAD VM
- Firewall rule is blocking the specific port
- Service requires credentials (not null session)

---

## Common Mistakes

- Editing /etc/hosts on the wrong machine (must be on Kali only)
- Using a tab character instead of spaces between IP and hostname
- Adding entries before verifying IPs against the live lab
- Forgetting that /etc/hosts changes take effect immediately (no restart needed)
- Assuming ping success means all services are reachable
- Using an FQDN that does not match exactly what is in /etc/hosts

---

## Rollback Procedure

If /etc/hosts is broken:

```
Execution Context: [KALI]
Shell: Bash
Privileges: root
```

```bash
# Restore from backup
sudo cp /etc/hosts.backup.<timestamp> /etc/hosts

# Verify localhost still resolves
getent hosts localhost
```

---

## Related Notes

- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Environment Setup and Validation]]
- [[GOAD - Error Audit and Contradiction Report]]
