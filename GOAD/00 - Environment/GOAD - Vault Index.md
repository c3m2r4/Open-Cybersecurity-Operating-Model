---
title: GOAD - Vault Index
category: Index
environment: GOAD
tags:
  - Index
  - GOAD
  - Mythic
  - Navigation
date_created: 2026-08-23
status: MAINTAINED
---

# GOAD Lab Vault — Index

> [!WARNING]
> This vault is for the **authorized GOAD lab / cybersecurity training environment only**.
> All commands are scoped to the 192.168.56.0/24 VirtualBox host-only network.

> [!CAUTION]
> **READ FIRST:** [[GOAD - Error Audit and Contradiction Report]]
> The original notes in this vault contain critical IP conflicts, wrong GitHub repos, and execution context errors.
> Do not use the old notes without reading the audit first.

---

## START HERE

1. [[GOAD - Error Audit and Contradiction Report]] — **Read before anything else**
2. [[GOAD - Command Execution Matrix]] — **Authoritative execution map (Phases 0–10)**
3. [[GOAD - Active Directory - Inside the Domain Workflow]] — **Post-compromise navigation map**
4. [[GOAD - Environment Setup and Validation]] — Verify lab is running
5. [[GOAD - IP Hostname Matrix]] — Verify all IPs
6. [[GOAD - Network Configuration and hosts Procedure]] — Configure /etc/hosts

---

## Vault Structure

```
Obsidian Vault/
├── GOAD/
│   ├── 00 - Environment/
│   │   ├── GOAD - Vault Index.md                    ← THIS FILE
│   │   ├── GOAD - Command Execution Matrix.md       ← AUTHORITATIVE EXECUTION MAP
│   │   ├── GOAD - Error Audit and Contradiction Report.md
│   │   └── GOAD - Environment Setup and Validation.md
│   ├── 01 - Network/
│   │   ├── GOAD - IP Hostname Matrix.md
│   │   └── GOAD - Network Configuration and hosts Procedure.md
│   ├── 02 - Enumeration/
│   │   ├── GOAD - Enumeration - Initial Recon.md
│   │   └── GOAD - Enumeration - Poisoning and Relay.md    [TODO — scaffold]
│   ├── 02 - Services/
│   │   └── GOAD - Services - MSSQL.md                     [TODO — scaffold]
│   ├── 03 - Active Directory/
│   │   ├── GOAD - Active Directory - BloodHound Collection.md
│   │   ├── GOAD - Active Directory - Inside the Domain Workflow.md
│   │   ├── GOAD - Active Directory - ADCS.md              [TODO — scaffold]
│   │   ├── GOAD - Active Directory - Delegation.md        [TODO — scaffold]
│   │   ├── GOAD - Active Directory - ACL and ACE Abuse.md [TODO — scaffold]
│   │   ├── GOAD - Active Directory - Trusts.md            [TODO — scaffold]
│   │   # TODO (not yet created): Kerberoasting, Domain Privilege Escalation
│   ├── 04 - Exploitation/
│   │   └── GOAD - Exploitation - Privilege Escalation.md  [TODO — framework only]
│   │   # TODO (not yet created): Initial Access Vectors
│   ├── 05 - Validation/
│   │   └── GOAD - Final Validation Checklist.md
│   └── 06 - Reporting/                                [TODO — folder not created]
│
├── Mythic/
│   ├── 00 - Architecture/
│   │   └── Mythic - Architecture Overview.md
│   ├── 01 - Installation/
│   │   └── Mythic - Installation and Verification.md
│   ├── 02 - Configuration/
│   │   └── Mythic - Payload Generation.md
│   ├── 03 - Agent Management/
│   │   └── (TODO)
│   ├── 04 - Lab Operations/
│   │   └── Mythic - Lab Operations - GodPotato and Mimikatz.md
│   └── 05 - Troubleshooting/
│       └── (TODO)
│
├── Operation Cypher-Knife/
│   ├── 00 - Scope/
│   │   └── Operation Cypher-Knife - Scope.md
│   ├── 01 - Objectives/                              [TODO — folder empty]
│   ├── 02 - Evidence/                                [TODO — folder empty]
│   ├── 03 - Findings/
│   │   └── Operation Cypher-Knife - Findings.md
│   └── 04 - Lessons Learned/                         [TODO — folder empty]
│
└── (vault root — OLD, DO NOT USE)
    ├── OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md
    ├── 🎯 MYTHIC C2 — COMPLETE OPERATION CYPHER-KNIFE RUNBOOK.md
    ├── 🏰 GOAD — GAME OF ACTIVE DIRECTORY — FULL ASSAULT PLAYBOOK.md
    └── 192.168.56.0 24 is the standard FULL GOAD network.md
```

> [!NOTE]
> Old notes are preserved for reference but should NOT be used for live operations without cross-checking against the Error Audit.

---

## Command Execution Context Reference

> Full phased procedures: [[GOAD - Command Execution Matrix]]

| Label | Meaning |
|---|---|
| `[KALI]` | Run on Kali Linux — Bash terminal |
| `[KALI-UI]` | Mythic or BloodHound web UI in browser on Kali |
| `[MYTHIC]` | Issue as task in Mythic UI to active callback |
| `[VICTIM-CMD]` | CMD on GOAD Windows host (Mythic `shell` task) |
| `[VICTIM-PS]` | PowerShell on GOAD Windows host (Mythic `powershell` task) |
| `[HOST]` | GOAD host machine where Vagrant runs (not Kali) |

---

## IP Quick Reference (VERIFY before use)

| Host | IP | Domain | Status |
|---|---|---|---|
| Kali | 192.168.56.1 (documented, `vmnet7`) | — | VERIFY before payload generation |
| King's Landing | 192.168.56.10 | sevenkingdoms.local | VERIFIED (2026-08-23 scan) |
| Winterfell | 192.168.56.11 | north.sevenkingdoms.local | VERIFIED (2026-08-23 scan) |
| Meereen | 192.168.56.12 | essos.local | VERIFIED (2026-08-23 scan) |
| Castle Black | 192.168.56.22 | north.sevenkingdoms.local | VERIFIED (2026-08-23 scan) |
| Braavos | 192.168.56.23 | essos.local | VERIFIED (2026-08-23 scan) |

Full details: [[GOAD - IP Hostname Matrix]]

---

## Original Notes Status

| Note | Status | Use? |
|---|---|---|
| `OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md` | Generic — wrong IPs (192.168.1.100) | Technique reference only |
| `🎯 MYTHIC C2 — COMPLETE OPERATION CYPHER-KNIFE RUNBOOK.md` | Partially GOAD-specific, placeholder URLs | Technique reference only |
| `GOAD — FULL ASSAULT PLAYBOOK.md` | GOAD-specific but wrong hostnames, wrong GitHub repo | Replace with new notes |
| `192.168.56.0 24 is the standard FULL GOAD network.md` | AI-generated, incorrect topology | Do not use |

---

## Final Validation Checklist

See [[GOAD - Final Validation Checklist]] for the complete pre-operation checklist.

---

## Note Inventory (Mayfly Coverage Audit — 2026-08-23)

### EXISTING (Maintained — procedures or validated content)

| Note | Role |
|---|---|
| [[GOAD - Command Execution Matrix]] | Authoritative Phases 0–10 |
| [[GOAD - Active Directory - Inside the Domain Workflow]] | Post-compromise navigation |
| [[GOAD - Environment Setup and Validation]] | Lab validation |
| [[GOAD - IP Hostname Matrix]] | Topology / IPs |
| [[GOAD - Network Configuration and hosts Procedure]] | /etc/hosts |
| [[GOAD - Enumeration - Initial Recon]] | Unauthenticated + pre-callback AD enum |
| [[GOAD - Active Directory - BloodHound Collection]] | SharpHound + BloodHound |
| [[GOAD - Final Validation Checklist]] | Pre-op checklist |
| [[GOAD - Error Audit and Contradiction Report]] | Known conflicts |
| [[Mythic - Installation and Verification]] | C2 setup |
| [[Mythic - Payload Generation]] | Stager delivery |
| [[Mythic - Lab Operations - GodPotato and Mimikatz]] | Local privesc + cred dump |
| [[Operation Cypher-Knife - Scope]] / [[Operation Cypher-Knife - Findings]] | Engagement tracking |

### NEW (Scaffold added — no fabricated procedures)

| Note | Purpose |
|---|---|
| [[GOAD - Services - MSSQL]] | MSSQL coverage checklist |
| [[GOAD - Active Directory - ADCS]] | ESC status matrix + discovery checklist |
| [[GOAD - Active Directory - Delegation]] | Delegation type coverage |
| [[GOAD - Active Directory - ACL and ACE Abuse]] | ACE/ACL concept coverage |
| [[GOAD - Active Directory - Trusts]] | Trust exploitation coverage |
| [[GOAD - Enumeration - Poisoning and Relay]] | Poisoning/relay concept coverage |
| [[GOAD - Exploitation - Privilege Escalation]] | Privesc category framework |

### TODO (Referenced but not yet created)

| Note | Referenced From |
|---|---|
| [[GOAD - Exploitation - Initial Access Vectors]] | Matrix Phase 3, Payload Generation |
| [[GOAD - Active Directory - Kerberoasting]] | Matrix P2-010, BloodHound note |
| [[GOAD - Active Directory - Domain Privilege Escalation]] | Matrix P8-003 |
| [[Operation Cypher-Knife - Lessons Learned]] | Scope, Findings |
| Mythic/03 - Agent Management/ | Matrix P4-009 |
| Mythic/05 - Troubleshooting/ | Matrix P4-009 |

### VERSION-DEPENDENT (Document when writing procedures)

| Technique | Source | Notes |
|---|---|---|
| Certifried (CVE-2022-26923) | Mayfly Part 6 | Patched on current AD — VERIFY |
| PetitPotam unauthenticated | Mayfly Parts 4, 6 | Patched — use authenticated coerce |
| ESC10 weak certificate mapping | Mayfly Part 14 | AD version dependent |
| Self-RBCD trick | Mayfly Part 10 | Reported silently patched — do not rely |
| Constrained delegation without S4U | Mayfly Part 10 | Requires lab tag — VERIFY |

### LAB-MODIFIED (Verify provisioning before use)

| Modification | Mayfly Source | Verify With |
|---|---|---|
| MSSQL impersonation / links | Part 7 | `ansible-playbook servers.yml` or live enum |
| ACL paths (ad-acl.yml) | Part 11 | BloodHound ACL collection |
| ADCS ESC4/5/7/9+ scenarios | Parts 6, 14 | certipy find / GOAD provision logs |
| SID History on trust | Part 12 | `ldeep ldap trusts` + trust flags |
| LLMNR/NBT-NS bots | Part 4 | Observe Responder traffic |
| WebClient on server (braavos) | Part 13 | CME webdav module |
| Constrained delegation kerb tag | Part 10 | findDelegation.py |

> External reference: [Mayfly GOAD category](https://mayfly277.github.io/categories/goad/) — gap analysis only.

---

## Missing Notes (TODO)

| Note | Referenced From |
|---|---|
| [[GOAD - Exploitation - Initial Access Vectors]] | Enumeration, Payload Generation, Command Matrix |
| [[GOAD - Active Directory - Kerberoasting]] | BloodHound note, Command Matrix P2-010 |
| [[GOAD - Active Directory - Domain Privilege Escalation]] | BloodHound note, Command Matrix P8-003 |
| [[Operation Cypher-Knife - Lessons Learned]] | Scope, Findings, Command Matrix |
| Mythic/03 - Agent Management/ | Vault Index, Command Matrix |
| Mythic/05 - Troubleshooting/ | Vault Index, Command Matrix |

> Scaffold notes (checklists only — procedures not written): [[GOAD - Services - MSSQL]], [[GOAD - Active Directory - ADCS]], [[GOAD - Active Directory - Delegation]], [[GOAD - Active Directory - ACL and ACE Abuse]], [[GOAD - Active Directory - Trusts]], [[GOAD - Enumeration - Poisoning and Relay]], [[GOAD - Exploitation - Privilege Escalation]]

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
- [[GOAD - Error Audit and Contradiction Report]]
- [[GOAD - Final Validation Checklist]]
- [[Mythic - Architecture Overview]]
- [[Operation Cypher-Knife - Scope]]
