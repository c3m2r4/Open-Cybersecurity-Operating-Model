---
title: GOAD - Error Audit and Contradiction Report
category: Audit
environment: GOAD
tags:
  - GOAD
  - Audit
  - Contradictions
  - IPConflict
date_created: 2026-08-23
status: REQUIRES-VERIFICATION
---

# GOAD — Error Audit & Contradiction Report

> [!CAUTION]
> This document must be reviewed before executing **any** commands in any other note in this vault.
> Every issue listed here has the potential to cause incorrect execution, wasted effort, or action against the wrong machine.

---

## Source Files Audited

| File | Size | Status |
|---|---|---|
| `192.168.56.0 24 is the standard FULL GOAD network.md` | 1.2 KB | Contradicts other notes |
| `OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md` | 80.7 KB | Generic — not GOAD-specific |
| `🎯 MYTHIC C2 — COMPLETE OPERATION CYPHER-KNIFE RUNBOOK.md` | 33.9 KB | Partially GOAD-specific |
| `🏰 GOAD — GAME OF ACTIVE DIRECTORY — FULL ASSAULT PLAYBOOK.md` | 28.9 KB | GOAD-specific but contradicts network note |

---

## ERROR 01 — Critical IP Address Conflict

> [!CAUTION] IP Conflict — STOP AND VERIFY
> **Two notes in this vault assign completely different IP addresses and hostnames to the same GOAD lab.**
>
> **Do not proceed until the deployed environment is verified.**

### Source A — /etc/hosts entries (provided in master prompt)

```text
192.168.56.10   kingslanding.sevenkingdoms.local   (King's Landing)
192.168.56.11   winterfell.north.sevenkingdoms.local
192.168.56.12   meereen.essos.local
192.168.56.22   castelblack.north.sevenkingdoms.local
192.168.56.23   braavos.essos.local
```

Domains: `sevenkingdoms.local`, `north.sevenkingdoms.local`, `essos.local`
Convention: Game of Thrones location names

### Source B — 192.168.56.0/24 network note + GOAD Assault Playbook

```text
192.168.56.2    Kali (attacker)
192.168.56.10   north-dc01    north.sevenkingdoms.local
192.168.56.11   north-mgmt    north.sevenkingdoms.local
192.168.56.12   north-web01   north.sevenkingdoms.local
192.168.56.20   child-dc01    child.sevenkingdoms.local
192.168.56.30   esdc01        essos.local
```

Domains: `north.sevenkingdoms.local`, `child.sevenkingdoms.local`, `essos.local`
Convention: Functional names (north-dc01, north-mgmt, etc.)

### Conflict Table

| IP | Source A (hosts file) | Source B (network note) | Match? |
|---|---|---|---|
| `192.168.56.10` | `kingslanding.sevenkingdoms.local` | `north-dc01.north.sevenkingdoms.local` | **NO** |
| `192.168.56.11` | `winterfell.north.sevenkingdoms.local` | `north-mgmt.north.sevenkingdoms.local` | **PARTIAL** |
| `192.168.56.12` | `meereen.essos.local` | `north-web01.north.sevenkingdoms.local` | **NO — different domain** |
| `192.168.56.20` | Not present | `child-dc01.child.sevenkingdoms.local` | UNKNOWN |
| `192.168.56.22` | `castelblack.north.sevenkingdoms.local` | Not present | UNKNOWN |
| `192.168.56.23` | `braavos.essos.local` | Not present | UNKNOWN |
| `192.168.56.30` | Not present | `esdc01.essos.local` | UNKNOWN |

> [!WARNING] Documentation Error Found
> **Original (network note):** `192.168.56.12` = `north-web01.north.sevenkingdoms.local`
>
> **Conflicting (/etc/hosts):** `192.168.56.12` = `meereen.essos.local`
>
> **Problem:** Same IP, completely different domain. Cannot both be correct.
>
> **Corrected:** VERIFY from live GOAD deployment only.
>
> **Reason:** The `192.168.56.0 24` note was AI-generated and does not reflect the actual GOAD topology.

> [!WARNING] Documentation Error Found
> **Original (network note):** `192.168.56.10` = `north-dc01.north.sevenkingdoms.local`
>
> **Conflicting (/etc/hosts):** `192.168.56.10` = `kingslanding.sevenkingdoms.local`
>
> **Problem:** The official GOAD project (Orange Cyberdefense) uses GoT location names. The "north-dc01" naming does not exist in the official topology.
>
> **Corrected:** VERIFY — the `/etc/hosts` GoT-name entries are more consistent with official GOAD.

---

## ERROR 02 — Mythic Server URL Not Populated

> [!WARNING] Documentation Error Found
> **Original:** `callback_host: https://mythic-server:443`
>
> **Problem:** `mythic-server` is a placeholder. Every stager using this URL will fail to call back.
>
> **Corrected:** Replace `mythic-server` with the actual Kali IP address. VERIFY actual Kali IP with `ip addr`.
>
> **Affected locations:** Mythic Runbook Phases 00.3, 01.1, 01.2, 01.3, 01.4, 06.2, 06.3, 06.4

---

## ERROR 03 — Duplicate Apollo Installation

> [!WARNING] Documentation Error Found
> **Original (GOAD Playbook Phase 00):**
> ```bash
> sudo ./mythic-cli install github https://github.com/its-a-feature/Apollo
> sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo
> ```
>
> **Problem:** Apollo installed twice from two different repos. Only `MythicAgents/Apollo` is correct.
>
> **Corrected:** `sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo`

---

## ERROR 04 — Missing Execution Context on `shell` Commands

> [!WARNING] Documentation Error Found
> **Original:** Commands like `shell hostname`, `shell whoami` throughout both GOAD and Mythic notes.
>
> **Problem:** `shell` is a Mythic agent task command — NOT a Kali Bash command.
>
> **Corrected:** All `shell <command>` lines must be issued inside the Mythic UI as tasks to an active Windows agent callback.
>
> **Affected:** GOAD Playbook Phases 03–13, Mythic Runbook Phases 02–13

---

## ERROR 05 — Mimikatz On-Disk Contradiction

> [!WARNING] Documentation Error Found
> **Original (Mythic Runbook Phase 06.1):** States "No mimikatz.exe on disk. Ever." then immediately shows a command that writes mimikatz.exe to C:\Windows\Temp\.
>
> **Problem:** Direct contradiction. The note itself catches this on the next line.
>
> **Corrected:** The hybrid script+upload method (Phase 06.3) is the correct approach. Mimikatz will briefly exist on disk — this is unavoidable when using GodPotato as a launcher.

---

## ERROR 06 — Responder Interface Hardcoded to eth0

> [!WARNING] Documentation Error Found
> **Original:** `sudo responder -I eth0 -wrf -d north.sevenkingdoms.local`
>
> **Problem:** `eth0` is hardcoded. The host-only interface name must be verified on the actual Kali machine.
>
> **Corrected:** Run `ip addr` first. Identify the interface with the 192.168.56.x address.

---

## ERROR 07 — Network Note Contains Non-GOAD Topology

> [!WARNING] Documentation Error Found
> **Original:** `192.168.56.0 24` note uses hostnames `north-dc01`, `north-mgmt`, `north-web01`, etc.
>
> **Problem:** These do NOT match the official GOAD topology. Official GOAD uses GoT names (KINGSLANDING, WINTERFELL, CASTELBLACK, MEEREEN, BRAAVOS).
>
> **Corrected:** The entire GOAD Assault Playbook was built on this incorrect note. All IP/hostname references in the playbook must be re-verified.

---

## ERROR 08 — execute-assembly Mimikatz Argument Syntax

> [!WARNING] Documentation Error Found
> **Original:** `execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:krbtgt" exit`
>
> **Problem:** Mythic's execute-assembly passes args to .NET Main() — not a shell. This syntax will not work as written.
>
> **Corrected:** Use the script file method or Invoke-Mimikatz via PowerShell in-memory.

---

## ERROR 09 — Cypher-Knife Runbook Uses Wrong IPs

> [!WARNING] Documentation Error Found
> **Original:** `OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md` uses `192.168.1.100` (attacker) and `10.0.0.5` (targets).
>
> **Problem:** These are NOT GOAD lab IPs. This is a generic technique reference, not a GOAD playbook.
>
> **Corrected:** Treat the Cypher-Knife runbook as a reference library only. Do not copy commands without replacing all IPs.

---

## ERROR 10 — Wrong Mythic GitHub Repository

> [!WARNING] Documentation Error Found
> **Original:** `git clone https://github.com/IT-007/Mythic.git`
>
> **Problem:** `IT-007/Mythic` does not exist. This will fail immediately.
>
> **Corrected:**
> ```bash
> git clone https://github.com/its-a-feature/Mythic.git
> cd Mythic
> sudo ./install_docker_ubuntu.sh
> sudo ./mythic-cli start
> ```

---

## Missing Information — Cannot Be Determined From Existing Notes

| Item | Status | How to Verify |
|---|---|---|
| Actual Kali interface name | VERIFY | `ip addr` |
| Actual Kali IP on 192.168.56.0/24 | VERIFY | `ip addr show <iface>` |
| GOAD uses GoT names or functional names | VERIFY | `nmap -sn 192.168.56.0/24`, then `nslookup <ip>` |
| Actual IP of KINGSLANDING / north-dc01 | VERIFY | nmap + nslookup |
| Actual IP of WINTERFELL / north-mgmt | VERIFY | nmap + nslookup |
| Actual IP of MEEREEN or north-web01 | VERIFY | nmap + nslookup |
| Whether child.sevenkingdoms.local exists | VERIFY | `nltest /domain_trusts` from domain member |
| Actual Mythic server IP/port | VERIFY | Depends on confirmed Kali IP |
| Whether GOAD VMs are running | VERIFY | `vagrant status` from GOAD directory |
| VirtualBox host-only adapter subnet | VERIFY | VirtualBox Host Network Manager |

---

## Final Consistency Matrix

| Check | Status |
|---|---|
| IPs consistent across all notes | FAIL — Major conflict |
| Hostnames consistent across all notes | FAIL — Two naming conventions |
| Mythic server URL populated | FAIL — Placeholder in use |
| Execution context labeled | FAIL — Most commands unlabeled |
| Privilege requirements documented | FAIL — SYSTEM assumed throughout |
| Mythic install command correct | FAIL — Wrong GitHub repo |
| Duplicate procedures identified | PARTIAL — Noted above |
| No fabricated information | PARTIAL — Default hashes unverified |

---

## Related Notes

- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Environment Setup and Validation]]
- [[GOAD - Network Configuration and hosts Procedure]]
- [[Mythic - Installation and Verification]]
