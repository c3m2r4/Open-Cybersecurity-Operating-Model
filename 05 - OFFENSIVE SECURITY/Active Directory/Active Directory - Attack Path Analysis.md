---
title: Active Directory - Attack Path Analysis
category: Security Domain
tags:
  - ActiveDirectory
  - AttackPath
  - RiskAnalysis
date_created: 2026-08-24
status: MAINTAINED
---

# Active Directory — Attack Path Analysis

## What Is an Attack Path?

An attack path is a sequence of actions an adversary can take — using legitimate access, misconfiguration, and technique — to move from an initial position to a high-value objective. Attack paths in Active Directory environments typically do not require zero-day exploitation. They rely on:

- Excessive privileges
- Misconfigured delegation
- ACL weaknesses
- Poor credential hygiene
- Weak monitoring

## Why Attack Path Analysis Matters

Fixing individual vulnerabilities in isolation may not reduce risk. An attacker does not need to exploit your most technically complex vulnerability — they will take the path of least resistance. Attack path analysis asks: **"Given what an attacker could realistically obtain, where can they go?"**

## Attack Path Structure

```text
Starting Position
  → Credential or Access Obtained
  → Enumeration of Available Paths
  → Next Hop (lateral movement or privilege escalation)
  → Enumeration of New Paths
  → Target (domain admin, Crown Jewels data, critical system)
```

Each hop in the path should be documented as a finding with its own business risk assessment.

## Example Path Types

### Path: Low-Privilege User to Domain Admin

| Step | Technique Concept | Weakness |
|---|---|---|
| 1 | User account compromised (phishing, credential stuffing) | No MFA; weak password |
| 2 | User enumerated domain users, groups, ACLs | LDAP accessible; broad query permissions |
| 3 | User found GenericWrite on IT service account | ACL misconfiguration |
| 4 | User reset service account password | GenericWrite enables password reset |
| 5 | Service account had Domain Admin membership | Excessive privilege |
| 6 | Domain Admin access obtained | Path complete |

**Business Risk:** Complete loss of domain control from a single compromised user account. No technical exploit required — only misconfiguration.

---

### Path: Kerberoasting to Lateral Movement

| Step | Technique Concept | Weakness |
|---|---|---|
| 1 | User requests Kerberos service ticket for SPN-registered service account | Any authenticated user can request service tickets |
| 2 | Encrypted ticket exported and cracked offline | Service account uses weak password |
| 3 | Service account credentials obtained | Weak password; no rotation |
| 4 | Service account used to access additional systems | Excessive service account permissions |
| 5 | Sensitive data or privileged access reached | No network segmentation; no monitoring |

**Business Risk:** Service accounts with weak passwords are exploitable by any domain-authenticated user. No elevated starting privilege required.

---

### Path: ADCS Misconfiguration to Domain Privilege

| Step | Technique Concept | Weakness |
|---|---|---|
| 1 | Standard user enumerates certificate templates | LDAP enumeration accessible |
| 2 | Misconfigured template found: allows client auth, user can enroll, SAN not restricted | Certificate template misconfiguration |
| 3 | User requests certificate claiming identity of domain admin | SAN override enables impersonation |
| 4 | Certificate used to authenticate as domain admin via PKINIT | Kerberos accepts certificate authentication |
| 5 | Domain Admin TGT obtained | Path complete |

**Business Risk:** A misconfigured certificate template can allow any authenticated user to obtain domain admin privileges. The attack leaves limited traditional footprint.

See: [[Active Directory Security - Overview]] — ADCS section; GOAD lab for controlled validation.

---

## Documenting Attack Paths as Findings

Each hop in an attack path should be evaluated:

| Field | Guidance |
|---|---|
| Starting Position | What access is required to begin this step? |
| Technique | What action is taken (in conceptual terms)? |
| Weakness | What misconfiguration or design flaw enables this? |
| Business Impact | If this hop succeeds, what risk is created? |
| Existing Controls | Is there a control that reduces likelihood or impact? |
| Detection | Would this step be detected? (link to detection notes) |
| Remediation | What change removes or significantly reduces this path? |

Full paths should be written as findings using [[Template - Professional Finding (Full)]].

---

## Attack Path Reduction Strategy

Attack paths are reduced by:

1. **Removing excessive privilege** — least privilege for users, groups, and service accounts
2. **Correcting ACL misconfigurations** — regular ACL audit; remove unnecessary write and control permissions
3. **Enforcing tiered administration** — credentials cannot traverse tiers
4. **Hardening ADCS** — restrict certificate templates; audit enrollment permissions
5. **Monitoring lateral movement** — detect pass-the-hash, pass-the-ticket, LDAP enumeration
6. **Credential hygiene** — enforce strong passwords for service accounts; use gMSA; rotate credentials
7. **Segmentation** — limit which systems a given account can reach even with valid credentials

See: [[Active Directory - Detection and Hardening]], [[Control - Network Segmentation]], [[Control - Privileged Access Management]]

---

## Lab Bridge

Attack path analysis is validated in the GOAD lab environment:

| Note | Purpose |
|---|---|
| [[GOAD Project - Risk and Control Bridge]] | Maps GOAD exploitation paths to business risk |
| [[GOAD - Command Execution Matrix]] | Authorized technical procedures |

**Do not reproduce GOAD command sequences here.** Use this note for analysis framework; use GOAD notes for execution.

---

## Related Notes

- [[Active Directory Security - Overview]] — foundational AD concepts
- [[Active Directory - Detection and Hardening]] — defensive response to attack paths
- [[Privilege Escalation - Concepts]] — privilege escalation techniques in context
- [[Lateral Movement - Concepts]] — movement between systems
- [[Risk-Based Offensive Security]] — how to assess finding severity
