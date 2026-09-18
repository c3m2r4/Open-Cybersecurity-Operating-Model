---
title: GOAD - Active Directory - Delegation
category: Active Directory
environment: GOAD
status: TODO
tags:
  - GOAD
  - Delegation
  - Kerberos
  - ActiveDirectory
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 10"
version_compatibility: GOAD v2 / LAB-MODIFIED / VERSION-DEPENDENT
---

# GOAD — Active Directory Delegation

> [!WARNING]
> **TODO — Procedures not yet written.** This note defines coverage requirements only.

> [!IMPORTANT]
> External reference: [Mayfly GOAD Part 10 — Delegations](https://mayfly277.github.io/posts/GOADv2-pwning-part10/)
> Mayfly demonstrates exploitation only; read delegation concepts from primary AD security resources before writing lab procedures.

---

## Delegation Types (Coverage Required)

| Type | Enumeration | Exploitation | Detection | Status |
|---|---|---|---|---|
| Unconstrained delegation | BloodHound query; `unconstraineddelegation:true` | TGT capture via coerce + Rubeus | Monitor unusual TGT requests | TODO |
| Constrained delegation (with protocol transition) | BloodHound; `findDelegation.py` | S4U2Self + S4U2Proxy (`getST.py`, Rubeus) | S4U audit events | TODO |
| Constrained delegation (without protocol transition) | Same | RBCD prerequisite chain — see below | Same | TODO — **LAB-MODIFIED** |
| Resource-Based Constrained Delegation (RBCD) | BloodHound; `msDS-AllowedToActOnBehalfOfOtherIdentity` | `addcomputer.py` + `rbcd.py` + `getST.py` | Computer account creation alerts | TODO |

---

## Coverage Checklist (To Document)

- [ ] Delegation enumeration from Kali (`findDelegation.py`, BloodHound)
- [ ] Delegation enumeration from callback (`setspn`, LDAP, BloodHound import)
- [ ] Unconstrained delegation validation (which hosts — DCs vs members)
- [ ] Constrained delegation validation (SPN, `msDS-AllowedToDelegateTo`)
- [ ] RBCD write permission validation
- [ ] Exploitation prerequisites per delegation type
- [ ] Cleanup (RBCD flush, computer account deletion)
- [ ] Detection considerations for lab documentation

---

## Version / Lab Compatibility

| Item | Classification | Notes |
|---|---|---|
| Unconstrained on child DC (WINTERFELL) | CURRENT GOAD | Default DC behavior — VERIFY |
| Constrained without protocol transition on CASTELBLACK | **LAB-MODIFIED** | Mayfly adds via `vulnerabilities.yml` tag `constrained_delegation_kerb` — **VERIFY before use** |
| Self-RBCD trick | **VERSION-DEPENDENT** | Mayfly reports Microsoft silent patch — do not document as reliable |
| RBCD via GenericWrite (stannis → kingslanding) | GOAD v2 | ACL path — see [[GOAD - Active Directory - ACL and ACE Abuse]] |

> [!WARNING]
> This technique requires a lab modification described by the external reference.
> Verify that constrained delegation without protocol transition exists in the current deployment before attempting the procedure.

---

## Relationship to Other Notes

| Escalation Path | Related Note |
|---|---|
| ACL-based RBCD setup | [[GOAD - Active Directory - ACL and ACE Abuse]] |
| NTLM relay → RBCD | [[GOAD - Enumeration - Poisoning and Relay]] |
| Trust escalation via unconstrained | [[GOAD - Active Directory - Trusts]] |
| BloodHound path discovery | [[GOAD - Active Directory - BloodHound Collection]] |

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Active Directory - ACL and ACE Abuse]]
- [[GOAD - Active Directory - Trusts]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
