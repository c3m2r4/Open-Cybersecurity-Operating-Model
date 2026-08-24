---
title: GOAD - Environment Setup and Validation
category: Environment
environment: GOAD
execution_context: "[KALI], [HOST], [MYTHIC], [VICTIM-CMD]"
tags:
  - GOAD
  - Environment
  - Setup
  - Validation
date_created: 2026-08-23
status: VERIFY-BEFORE-USE
---

# GOAD — Environment Setup & Validation

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.
> Do not run these procedures against any system you do not own or have explicit authorization to test.

> [!IMPORTANT]
> Read [[GOAD - Error Audit and Contradiction Report]] before starting any procedure in this vault.
> Critical IP conflicts exist between existing notes.

---

## What is GOAD?

GOAD (Game of Active Directory) is an intentionally vulnerable Active Directory environment by Orange Cyberdefense.
Repository: https://github.com/Orange-Cyberdefense/GOAD

It provides a safe, legal environment to practice Active Directory attacks, lateral movement, and credential abuse.

---

## Prerequisites

- [ ] VirtualBox installed and configured
- [ ] Vagrant installed
- [ ] GOAD repository cloned
- [ ] Host-only network adapter configured for 192.168.56.0/24
- [ ] At least 12 GB RAM available (5 VMs)
- [ ] Kali Linux VM configured on the host-only adapter
- [ ] Mythic C2 installed on Kali (see [[Mythic - Installation and Verification]])

---

## Step 1 — Verify GOAD VMs Are Running

```
Execution Context: HOST MACHINE (not Kali)
Shell: Bash (host terminal)
Privileges: Normal user
```

```bash
# Navigate to your GOAD installation directory
cd /path/to/GOAD

# Check status of all VMs
vagrant status
```

**Expected result:**
```
Current machine states:

dc01          running (virtualbox)
dc02          running (virtualbox)
dc03          running (virtualbox)
srv02         running (virtualbox)
srv03         running (virtualbox)
```

> [!NOTE]
> VM names may differ depending on your GOAD version (GOAD, GOAD-Light, etc.).
> The standard GOAD has 5 VMs.

If VMs are not running:
```bash
vagrant up
```

---

## Step 2 — Verify Kali's Network Position

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# List all network interfaces
ip addr

# Identify the interface with a 192.168.56.x address
# Common names: eth0, eth1, vboxnet0
ip addr show | grep "192.168.56"
```

**Expected result:** One interface shows an IP in the 192.168.56.0/24 range.

**Record your Kali IP here:** `VERIFY — run ip addr`

---

## Step 3 — Verify Network Reachability

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Sweep for live hosts
sudo nmap -sn 192.168.56.0/24

# Check routing
ip route show
# Should show: 192.168.56.0/24 dev <interface>
```

**Expected result:** 5–6 hosts respond (GOAD VMs + Kali itself).

---

## Step 4 — Verify /etc/hosts Configuration

Run this ONLY after completing [[GOAD - Network Configuration and hosts Procedure]]:

```bash
getent hosts kingslanding
getent hosts winterfell
getent hosts meereen
getent hosts castelblack
getent hosts braavos
```

---

## Step 5 — Verify SMB Connectivity (Quick Domain Check)

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Null session test to King's Landing (sevenkingdoms.local DC)
# Replace IP with verified IP from [[GOAD - IP Hostname Matrix]]
smbclient -L //192.168.56.10 -N 2>&1 | head -20
```

**Expected result:** Share list is returned (even if empty or with limited shares).

**If it fails:**
- VM not running
- Firewall blocking SMB (port 445)
- Wrong IP

---

## Step 6 — Verify Domain Controller Responsiveness

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Check if Kerberos port is open (88/tcp = DC is up and accepting auth)
nmap -p 88,389,445,3389 192.168.56.10
nmap -p 88,389,445,3389 192.168.56.11
nmap -p 88,389,445,3389 192.168.56.12
nmap -p 88,389,445 192.168.56.22
nmap -p 88,389,445 192.168.56.23
```

**Expected result:** Ports 445 open on all. Port 88 open on DCs (King's Landing, Winterfell, Meereen).

---

## Environment Validation Checklist

- [ ] GOAD VMs are running (`vagrant status`)
- [ ] Kali has an IP on 192.168.56.0/24 (verify with `ip addr`)
- [ ] All GOAD hosts respond to ICMP (`ping -c 2 <ip>`)
- [ ] /etc/hosts configured with verified IPs
- [ ] Hostname resolution verified with `getent hosts`
- [ ] SMB reachable on at least one DC
- [ ] Kerberos port (88) open on DCs
- [ ] Mythic server running (see [[Mythic - Installation and Verification]])

---

## Common Mistakes

- Running GOAD commands from the host machine instead of Kali
- Forgetting to start the GOAD VMs before attempting connections
- Using /etc/hosts entries that were copied without verification
- Running enumeration tools before confirming basic connectivity
- Assuming Kali is always at 192.168.56.2 — it may differ by installation

---

## Related Notes

- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Network Configuration and hosts Procedure]]
- [[GOAD - Error Audit and Contradiction Report]]
- [[Mythic - Installation and Verification]]
