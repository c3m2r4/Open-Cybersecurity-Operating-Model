---
title: GOAD - Active Directory - Inside the Domain Workflow
category: Active Directory
environment: GOAD
status: MAINTAINED
tags:
  - GOAD
  - Workflow
  - Navigation
  - PostExploitation
date_created: 2026-08-23
---

# GOAD — Inside the Domain Workflow

> [!IMPORTANT]
> Navigational map for post-compromise operations. **Does not duplicate commands** — links to detailed notes and matrix procedures.
> Complete Phase 0 before starting. Follow [[GOAD - Command Execution Matrix]] for authoritative procedure IDs.

---

## Workflow

```text
Initial foothold          → Phase 3: P3-003–P3-005 | [[Mythic - Payload Generation]]
      ↓
Identity validation       → Phase 3: P3-005 | Phase 8: P8-001 | [[GOAD - Final Validation Checklist]] Section H
      ↓
Domain enumeration        → Phase 2: P2-006–P2-009 (post-callback) | [[GOAD - Active Directory - BloodHound Collection]]
      ↓
Privilege / path discovery → BloodHound queries | TODO: [[GOAD - Active Directory - ACL and ACE Abuse]], [[GOAD - Active Directory - Delegation]]
      ↓
Credential / access expansion → Phase 6: P6-001–P6-005 | Phase 2: P2-010 (Kerberoast — TODO)
      ↓
Lateral movement          → Phase 7: P7-001–P7-003 | TODO: MSSQL links, WinRM, PsExec paths
      ↓
Domain-level objectives   → Phase 8 | TODO: [[GOAD - Active Directory - Domain Privilege Escalation]]
      ↓
Cross-domain analysis     → P8-002 | TODO: [[GOAD - Active Directory - Trusts]]
      ↓
Evidence                  → Phase 9: P9-001 | [[Operation Cypher-Knife - Findings]]
      ↓
Cleanup                   → Phase 10: P10-001–P10-004
```

---

## Phase → Note Map

| Workflow Stage | Matrix Procedures | Detailed Notes |
|---|---|---|
| Initial foothold | P3-001–P3-005 | [[Mythic - Payload Generation]], TODO: [[GOAD - Exploitation - Initial Access Vectors]] |
| Unauthenticated recon | P1-001–P1-006, P2-001–P2-005 | [[GOAD - Enumeration - Initial Recon]] |
| Poisoning / relay (optional) | *Not in matrix* | TODO: [[GOAD - Enumeration - Poisoning and Relay]] |
| Authenticated enum (pre-callback) | P2-001–P2-005 | [[GOAD - Enumeration - Initial Recon]] |
| Mythic operations | P4-001–P4-009 | [[Mythic - Architecture Overview]] |
| Local privesc (Windows) | P5-001–P5-002 | [[Mythic - Lab Operations - GodPotato and Mimikatz]], TODO: [[GOAD - Exploitation - Privilege Escalation]] |
| Credential access | P6-001–P6-005 | [[Mythic - Lab Operations - GodPotato and Mimikatz]] |
| Lateral movement | P7-001–P7-003 | Matrix only (PtH, SOCKS) — gaps: MSSQL, RDP, WinRM |
| ADCS paths | *Not in matrix* | TODO: [[GOAD - Active Directory - ADCS]] |
| MSSQL paths | *Not in matrix* | TODO: [[GOAD - Services - MSSQL]] |
| Delegation paths | *Not in matrix* | TODO: [[GOAD - Active Directory - Delegation]] |
| ACL / ACE paths | *Not in matrix* | TODO: [[GOAD - Active Directory - ACL and ACE Abuse]] |
| Trust paths | P8-002 only | TODO: [[GOAD - Active Directory - Trusts]] |
| Domain compromise | P8-001–P8-003 | [[Operation Cypher-Knife - Scope]] |
| Inside-domain coerce (authenticated) | *Not in matrix* | Mayfly Part 13 — TODO note TBD |

---

## Privilege Escalation Categories

When [[GOAD - Exploitation - Privilege Escalation]] is written, it must distinguish:

| Category | Current Coverage | Note |
|---|---|---|
| Windows local privilege escalation | Partial | GodPotato (P5-002), SharpUp (P4-002) |
| Active Directory privilege escalation | Missing | P8-003 TODO |
| Delegation-based escalation | Missing | [[GOAD - Active Directory - Delegation]] |
| ACL-based escalation | Missing | [[GOAD - Active Directory - ACL and ACE Abuse]] |
| Credential-based escalation | Partial | AS-REP (P2-004), Mimikatz (P6-001), PtH (P7-001) |
| Cross-domain escalation | Missing | [[GOAD - Active Directory - Trusts]] |

---

## External Coverage Reference

Compared against [Mayfly GOAD series](https://mayfly277.github.io/categories/goad/) (17 posts).
See [[GOAD - Command Execution Matrix#External Coverage Reference (Mayfly)]] for full gap table.

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Vault Index]]
- [[Operation Cypher-Knife - Scope]]
- [[Operation Cypher-Knife - Findings]]
