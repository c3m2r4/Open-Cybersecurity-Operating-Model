---
title: GOAD - Active Directory - Trusts
category: Active Directory
environment: GOAD
status: TODO
tags:
  - GOAD
  - Trusts
  - ActiveDirectory
  - CrossDomain
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 12; Part 7 (MSSQL links)"
version_compatibility: GOAD v2 / LAB-MODIFIED
---

# GOAD — Active Directory Trusts

> [!WARNING]
> **TODO — Procedures not yet written.** Trust exploitation paths vary by deployment configuration.
> Do not assume SID History, foreign groups, or MSSQL links exist without verification.

> [!IMPORTANT]
> External reference: [Mayfly GOAD Part 12 — Trusts](https://mayfly277.github.io/posts/GOADv2-pwning-part12/)
> Documented topology (VERIFY live): parent `sevenkingdoms.local`, child `north.sevenkingdoms.local`, forest `essos.local`.

---

## Documented Trust Topology (VERIFY)

From [[GOAD - IP Hostname Matrix]] — not independently verified in this note:

```text
sevenkingdoms.local (root)
  └── north.sevenkingdoms.local (child, bidirectional)

essos.local (separate forest, external trust to sevenkingdoms.local)
```

---

## Coverage Checklist (To Document)

| Area | Current Vault | Status |
|---|---|---|
| Parent/child domain relationship | IP Matrix diagram only | Partial |
| Forest trust relationship | IP Matrix diagram only | Partial |
| Trust enumeration (`nltest`, ldeep, BloodHound) | P2-006, P8-002 — basic only | Partial |
| Foreign users/groups (cross-domain membership) | Not documented | **Missing** |
| SIDHistory considerations | Not documented | **Missing** |
| Child → parent escalation (raiseChild, golden ticket + ExtraSid) | Not in maintained vault | **Missing** |
| Inter-realm trust ticket forgery | Not in maintained vault | **Missing** |
| Forest → forest (password reuse, foreign groups) | Not in maintained vault | **Missing** |
| Unconstrained delegation cross-forest | Mentioned in Mayfly; links to [[GOAD - Active Directory - Delegation]] | TODO |
| MSSQL trusted links (cross-forest) | Not documented — see [[GOAD - Services - MSSQL]] | **Missing** |
| Trust-related BloodHound analysis | Map Domain Trusts button — not procedured | Partial |
| Golden ticket + SID History (external forest) | **LAB-MODIFIED** — Mayfly adds SID History via `vulnerabilities.yml` | TODO — VERIFY |

---

## Existing Matrix Coverage

| Procedure | What It Covers |
|---|---|
| P2-006 | `nltest /domain_trusts` from callback |
| P8-002 | Trust verification against IP Matrix diagram |

These are **enumeration only** — not exploitation paths.

---

## Version / Lab Compatibility

| Item | Classification | Action |
|---|---|---|
| Basic child/parent trust | CURRENT GOAD | VERIFY with P8-002 |
| DragonRider group + SID History on trust | **LAB-MODIFIED** | Run Mayfly ansible provisions or VERIFY manually |
| Foreign group Spys / AcrossTheNarrowSea | **LAB-MODIFIED** | VERIFY group membership in live AD |
| MSSQL link CASTELBLACK → BRAAVOS | GOAD v2 — **VERIFY** | See [[GOAD - Services - MSSQL]] |

> [!WARNING]
> This technique requires a lab modification described by the external reference.
> Verify that SID History and foreign-group ACL paths exist in the current deployment before attempting cross-forest escalation.

---

## Prerequisites (When Written)

- [ ] P2-006 / P8-002 trust enumeration complete
- [ ] BloodHound trust map imported — [[GOAD - Active Directory - BloodHound Collection]]
- [ ] Domain credentials for owned domain — [[Operation Cypher-Knife - Findings]]
- [ ] Trust type and direction confirmed (not assumed from diagram)

---

## Related Notes

- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Command Execution Matrix]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Active Directory - Delegation]]
- [[GOAD - Active Directory - ACL and ACE Abuse]]
- [[GOAD - Services - MSSQL]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
