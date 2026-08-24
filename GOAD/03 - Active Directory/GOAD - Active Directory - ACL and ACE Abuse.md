---
title: GOAD - Active Directory - ACL and ACE Abuse
category: Active Directory
environment: GOAD
status: TODO
tags:
  - GOAD
  - ACL
  - ACE
  - DACL
  - ActiveDirectory
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 11"
version_compatibility: GOAD v2 / LAB-MODIFIED
---

# GOAD — Active Directory ACL and ACE Abuse

> [!WARNING]
> **TODO — Procedures not yet written.** ACL abuse **modifies the lab environment**.
> Every procedure must include validation, evidence, and cleanup steps before use.

> [!CAUTION]
> Mayfly Part 11 requires lab ACL updates (`ad-data.yml`, `ad-acl.yml`, `ad-relations.yml`, `vulnerabilities.yml`).
> AdminSDHolder resets protected-group ACLs hourly — paths targeting Domain Admins may not persist.
> **Verify ACL paths against live BloodHound collection; do not assume Mayfly paths exist.**

> [!IMPORTANT]
> External reference: [Mayfly GOAD Part 11 — ACL](https://mayfly277.github.io/posts/GOADv2-pwning-part11/)

---

## ACE / ACL Concepts (Coverage Required)

| Permission | Target Types | Abuse Techniques | Modifies Lab? | Status |
|---|---|---|---|---|
| GenericAll | User, Group, Computer | ForceChangePassword, RBCD, Shadow Credentials, Kerberoast | **Yes** | TODO |
| GenericWrite | User | Target Kerberoasting, Shadow Credentials, logon script, profilePath coerce | **Yes** | TODO |
| WriteDACL | User, Group | Grant self FullControl → chain | **Yes** | TODO |
| WriteOwner | Group | Take ownership → grant self rights | **Yes** | TODO |
| ForceChangePassword | User | Password reset | **Yes** — destructive | TODO — document as last resort |
| AddMember / Self | Group | Group membership escalation | **Yes** | TODO |
| GenericAll on Computer | Computer | RBCD, Shadow Credentials on `<LAB_HOST>$` | **Yes** | TODO |
| Write on msDS-KeyCredentialLink | User/Computer | Shadow Credentials (requires ADCS) | **Yes** | TODO — see [[GOAD - Active Directory - ADCS]] |
| LAPS read permission | Computer | Read local admin password | No (read) | TODO |
| GPO abuse (Edit settings) | GPO | Scheduled task / user creation | **Yes** | TODO |

---

## Protected Groups / AdminSDHolder

Document in procedures:

- [ ] List of AdminSDHolder-protected groups (Domain Admins, Enterprise Admins, etc.)
- [ ] Why ACL paths to protected groups may disappear after ~60 minutes
- [ ] BloodHound query to find non-protected ACL paths: `(u)-[r:ACL]->(n) WHERE u.admincount=false`

---

## BloodHound Integration

| Query / Feature | Purpose | Status |
|---|---|---|
| Shortest path to Domain Admin | Primary path discovery | Partial — [[GOAD - Active Directory - BloodHound Collection]] |
| ACL edges | Path visualization | Partial — collection exists; abuse procedures TODO |
| Kerberoastable via GenericWrite | Target Kerberoasting | TODO — [[GOAD - Active Directory - Kerberoasting]] |
| Shadow Credentials edges | ADCS + Write | TODO — [[GOAD - Active Directory - ADCS]] |

---

## Version / Lab Compatibility

| Item | Classification |
|---|---|
| sevenkingdoms.local ACL kill chain (Tywin → kingslanding) | GOAD v2 — **VERIFY live** |
| GPO abuse on north domain | GOAD v2 — **VERIFY live** |
| LAPS on essos.local | GOAD v2 — **VERIFY** ADCS + LAPS module |
| ACL refresh via ansible | **LAB-MODIFIED** |

---

## Procedure Requirements (When Written)

Each ACL abuse procedure must document:

```text
Execution Context: [KALI] or [MYTHIC] → [VICTIM-CMD/PS]
Source identity: <LAB_USER>
Target object: <LAB_USER> / <LAB_HOST> / group
Permission used: GenericAll / GenericWrite / etc.
Validation: How to confirm success without assuming creds
Evidence: Record in [[Operation Cypher-Knife - Findings]]
Cleanup: ACL rollback / password restore / group removal / GPO revert
Detection: What AD event or artifact is created
```

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Active Directory - Delegation]]
- [[GOAD - Active Directory - ADCS]]
- [[GOAD - Active Directory - Domain Privilege Escalation]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
