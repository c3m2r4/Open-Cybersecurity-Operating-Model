---
title: GOAD - Final Validation Checklist
category: Validation
environment: GOAD
tags:
  - GOAD
  - Validation
  - Checklist
  - PreOperation
date_created: 2026-08-23
status: MAINTAINED
---

# GOAD — Final Validation Checklist

> [!IMPORTANT]
> Complete this checklist before starting any active exploitation phase.
> Every unchecked item represents a potential failure point that will cost time to debug mid-operation.

---

## Section A — Error Audit Review

- [ ] [[GOAD - Error Audit and Contradiction Report]] has been read in full
- [ ] All ERROR items are understood
- [ ] Old notes are not being used as primary references

---

## Section B — GOAD Hosts Verified

Verify each host from Kali:

```bash
sudo nmap -sn 192.168.56.0/24
```

- [ ] Host at 192.168.56.10 responds and hostname verified
- [ ] Host at 192.168.56.11 responds and hostname verified
- [ ] Host at 192.168.56.12 responds and hostname verified
- [ ] Host at 192.168.56.22 responds and hostname verified
- [ ] Host at 192.168.56.23 responds and hostname verified

Record verified hostnames in [[GOAD - IP Hostname Matrix]].

---

## Section C — IP Addresses Verified

- [ ] IPs confirmed by `nmap -sn` and `nslookup` (not just from documentation)
- [ ] No IP conflicts between notes remain unresolved
- [ ] [[GOAD - IP Hostname Matrix]] updated with verified values
- [ ] Kali IP on 192.168.56.0/24 confirmed with `ip addr`

---

## Section D — DNS Resolution Verified

```bash
getent hosts kingslanding.sevenkingdoms.local
getent hosts winterfell.north.sevenkingdoms.local
getent hosts meereen.essos.local
getent hosts castelblack.north.sevenkingdoms.local
getent hosts braavos.essos.local
```

- [ ] All GOAD hostnames resolve to correct IPs
- [ ] /etc/hosts backed up before modification
- [ ] nsswitch.conf has `files` before `dns`

---

## Section E — Network Routing Verified

```bash
ip route show | grep "192.168.56"
ping -c 2 192.168.56.10
ping -c 2 192.168.56.11
ping -c 2 192.168.56.12
```

- [ ] 192.168.56.0/24 route exists on Kali
- [ ] All GOAD hosts respond to ICMP ping
- [ ] SMB port (445) reachable on at least one DC

---

## Section F — Kali Configuration Verified

- [ ] Kali interface on 192.168.56.0/24 identified (run `ip addr`)
- [ ] Kali IP recorded: `VERIFY`
- [ ] /etc/hosts correct and backed up
- [ ] Required tools installed: nmap, smbclient, crackmapexec, impacket, bloodhound

---

## Section G — Mythic Infrastructure Verified

- [ ] Mythic Docker containers healthy (`sudo docker ps`)
- [ ] Mythic UI accessible at `https://127.0.0.1:7443`
- [ ] Apollo agent installed and visible in Mythic UI
- [ ] HTTP C2 profile installed
- [ ] Supporting binaries uploaded to Mythic File Host (GodPotato, Mimikatz, SharpHound, Rubeus, Seatbelt)
- [ ] Mythic payload generated with correct Kali IP (NOT `mythic-server` placeholder)

---

## Section H — Windows Hosts Verified

From initial callback in Mythic (after first stager delivery):

```
Execution Context: [MYTHIC] → [VICTIM-CMD]
Task type: shell (issued in Mythic UI — NOT in Kali terminal)

shell whoami
shell hostname
shell ipconfig
shell nltest /domain_trusts
```

- [ ] Callback hostname matches expected GOAD VM name
- [ ] Domain matches expected value from [[GOAD - IP Hostname Matrix]]
- [ ] Current user context is documented

---

## Section I — Domain Information Verified

```
Execution Context: [MYTHIC] → [VICTIM-CMD]
Task type: shell (issued in Mythic UI — NOT in Kali terminal)

shell nltest /domain_trusts
shell net group "Domain Admins" /domain
```

- [ ] Domain names confirmed (sevenkingdoms.local, north.sevenkingdoms.local, essos.local)
- [ ] Trust relationships confirmed
- [ ] Domain Admin group membership enumerated

---

## Section J — Commands Assigned to Correct Execution Contexts

Before each operation:
- [ ] Every command has a documented execution context label
- [ ] No `shell` commands are being run in Kali terminal (they go to Mythic)
- [ ] No Kali Bash commands are being issued as Mythic tasks
- [ ] Bash and PowerShell syntax are not mixed

---

## Section K — Privilege Requirements Documented

- [ ] Every privilege-requiring command has `Privileges: root` or `Privileges: Administrator` labeled
- [ ] `sudo` is explicit where required
- [ ] GodPotato privilege requirement (SeImpersonatePrivilege) verified before attempting

---

## Section L — Prerequisites Documented

- [ ] Each operation section has a Prerequisites checklist
- [ ] No step is executed before its prerequisite is confirmed
- [ ] GodPotato is tested BEFORE attempting credential dumping

---

## Section M — Expected Results Documented

- [ ] Each operation section documents what success looks like
- [ ] Actual observed results are being recorded (not assumed)
- [ ] Failures are documented with observed output

---

## Section N — Troubleshooting Documented

- [ ] "If it fails" sections exist for each major procedure
- [ ] Common mistakes are listed

---

## Section O — Duplicate Notes Consolidated

- [ ] Old runbook notes are marked as `[OLD — DO NOT USE]` or archived
- [ ] No procedure is documented in two places without cross-linking
- [ ] New notes link to each other correctly

---

## Section P — No Fabricated Information Remains

- [ ] No GOAD default hashes are being used without verifying against actual lab
- [ ] No domain names are assumed without verification
- [ ] No IPs are hardcoded from old notes without verification
- [ ] `VERIFY` placeholder is used where information is unknown

---

## Pre-Exploitation Sign-Off

Complete this section before beginning active exploitation:

```
Date: VERIFY
GOAD environment status: RUNNING / NOT RUNNING
Kali IP: VERIFY
Mythic server status: RUNNING / NOT RUNNING
All hosts reachable: YES / NO (list exceptions)
Error Audit reviewed: YES / NO
Notes used: [list which new notes you are using]
Operator: VERIFY
```

---

## Related Notes

- [[GOAD - Vault Index]]
- [[GOAD - Error Audit and Contradiction Report]]
- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Environment Setup and Validation]]
- [[Mythic - Installation and Verification]]
