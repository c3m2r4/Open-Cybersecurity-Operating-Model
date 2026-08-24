---
title: Offensive Security Safety and Authorization
category: Methodology
tags:
  - OffensiveSecurity
  - Authorization
  - Safety
  - Ethics
date_created: 2026-08-24
status: MAINTAINED
---

# Offensive Security Safety and Authorization

## The Absolute Rule

> **No testing activity — no scan, no probe, no exploit attempt — begins without explicit written authorization from the asset owner.**

Authorization is not optional. It is the legal, ethical, and professional foundation of all offensive security work. Testing systems without authorization is unauthorized access, regardless of intent, skill level, or perceived benefit.

---

## Three Operating Contexts

Understanding which context you are operating in determines what is permitted, what evidence is required, and what level of care must be applied.

### LAB

| Field | Detail |
|---|---|
| **Definition** | An isolated environment that you own or control, built specifically for learning, testing, and development |
| **Examples** | GOAD (local VM lab), Mythic lab, personal virtual machines, intentionally vulnerable applications |
| **Authorization** | You are the owner and operator — no external authorization is required |
| **Limitations** | Actions must remain within the isolated environment; lab traffic must not egress to production networks or the internet in uncontrolled ways |
| **Purpose** | Learning techniques, validating tools, building detection capability, practicing methodology |
| **Evidence Standard** | Lab notes with scope, actions, observations, and cleanup |

> LAB activity builds skills. It does not produce professional findings that can be claimed as representing a real environment's risk.

See: [[GOAD Project - Risk and Control Bridge]], [[Mythic Lab - Project Bridge]]

---

### AUTHORIZED ASSESSMENT

| Field | Detail |
|---|---|
| **Definition** | Testing of a real environment under written authorization from the asset owner |
| **Examples** | Penetration test, vulnerability assessment, red team engagement, purple team exercise, control validation |
| **Authorization** | Written authorization signed by appropriate asset owner; scope documented; ROE agreed |
| **Limitations** | Scope strictly defines what is in scope; out-of-scope systems must not be tested even if discovered; production risk must be managed per agreed rules |
| **Purpose** | Produce professional findings, control assessments, and business risk evidence |
| **Evidence Standard** | Authorization document; timestamped logs; finding evidence; cleanup confirmation |

> AUTHORIZED ASSESSMENT activity produces findings that can be used to inform risk decisions. Without authorization, findings have no professional standing.

---

### PRODUCTION

| Field | Detail |
|---|---|
| **Definition** | A live, operational environment that supports real business processes or customer services |
| **Special Considerations** | Any disruption has direct business impact; heightened care required for all actions; some techniques that are safe in lab environments are not safe in production |
| **Authorization** | Authorization must explicitly address production; confirm with operations team before each phase |
| **Techniques to Confirm Explicitly** | DoS-adjacent techniques, credential brute force, network scanning intensity, payload execution, persistence establishment |
| **Evidence** | Timestamped log of every action; emergency stop contacts active; operations team notified per ROE |
| **Emergency Stop** | If unexpected impact is observed, stop immediately, notify the emergency contact, and document the event |

> When in doubt in production: **stop, document, and ask.** It is always better to pause and confirm than to cause an outage.

---

## Authorization Requirements

### Minimum Requirements for Any Authorized Assessment

- [ ] Written authorization from the asset owner (not just a verbal agreement)
- [ ] Named in-scope systems, applications, or network ranges
- [ ] Explicit out-of-scope systems or areas
- [ ] Agreed testing window or schedule
- [ ] Emergency contact information (who to call if something goes wrong)
- [ ] Data handling agreement (how sensitive data discovered during testing is handled)
- [ ] Cleanup confirmation requirement

### Additional Requirements for Specific Engagement Types

| Engagement Type | Additional Requirements |
|---|---|
| Red Team | Executive-level authorization; SOC must not be told; crisis communication plan |
| Production Active Directory | Change management coordination; backup confirmation; domain admin emergency contact |
| Cloud Environments | Cloud provider notification (some providers require prior notification for security testing on shared infrastructure) |
| Regulated Environments | Confirm regulatory obligations around testing and evidence handling — VERIFY per jurisdiction |
| Third-Party Systems | Separate authorization from the third party that owns the system |

---

## Rules of Engagement — Standard Elements

A Rules of Engagement document should address:

| Element | Purpose |
|---|---|
| Permitted Hours | Reduce risk of testing during critical business operations |
| Source IP Addresses | Ensure defensive teams can filter test traffic if needed; confirm tester activity |
| Notification Triggers | What constitutes a critical finding requiring immediate notification |
| Emergency Stop Criteria | Conditions that require immediate cessation of all testing |
| Communication Channel | Primary and backup contact method during testing |
| Data Handling | How discovered credentials, PII, or sensitive data must be handled |
| Evidence Handling | How evidence is stored, transmitted, and destroyed after the engagement |
| Third-Party Systems | Explicit list of systems that require separate authorization |
| Production Constraints | Specific techniques prohibited or constrained in production |

---

## What Happens Without Authorization

Using offensive security techniques against systems you do not own or have explicit permission to test is:

- Unauthorized access — a criminal offence in most jurisdictions
- A professional ethics violation
- A reputational and legal risk regardless of your intent or skill

Being a security professional does not create an implicit right to test systems. Even testing a client's system without properly documented authorization creates legal exposure for both the tester and the client.

---

## Red Flags — Stop and Confirm

Stop and seek written clarification if any of the following occur:

- You have reached an asset that was not explicitly named in scope
- You have discovered a third-party provider's infrastructure while testing in scope
- You have discovered a system that appears to be regulated (healthcare data, financial records, critical infrastructure)
- Testing is producing unexpected system behaviour
- You have obtained credentials that appear to be for real users or high-privilege accounts
- A system appears to have connectivity to out-of-scope environments

---

## Related Notes

- [[Offensive Security Methodology]] — Phase 1 (Authorization), Phase 3 (Rules of Engagement), Phase 13 (Cleanup)
- [[Security Testing Types]] — authorization requirements differ by testing type
- [[GOAD Project - Risk and Control Bridge]] — lab environment context
- [[Mythic Lab - Project Bridge]] — lab environment context
- [[VAPT - Project Bridge]] — assessment methodology bridge
