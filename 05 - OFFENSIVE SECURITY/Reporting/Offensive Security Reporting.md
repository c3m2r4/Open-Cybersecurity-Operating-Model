---
title: Offensive Security Reporting
category: Methodology
tags:
  - OffensiveSecurity
  - Reporting
  - Findings
  - RiskCommunication
date_created: 2026-08-24
status: MAINTAINED
---

# Offensive Security Reporting

## Purpose

A finding is only valuable if it leads to action. Offensive security reporting converts technical evidence into risk-based findings that can be understood, prioritized, and acted upon by technical, risk, and management audiences.

> A report that demonstrates technical sophistication but does not clearly communicate business risk, root cause, and remediation is not a professional finding — it is a technical log.

---

## The Professional Finding Template

Every finding should contain the following fields. See [[Template - Professional Finding (Full)]] for the template.

| Field | Purpose |
|---|---|
| **Title** | Clear, concise description of what was found |
| **Finding ID** | Unique identifier for tracking |
| **Severity** | Risk-based rating (Critical / High / Medium / Low / Informational) |
| **Affected Asset** | Specific system, application, or environment affected |
| **Business Context** | What business process or objective does this asset support? |
| **Description** | Clear explanation of the weakness — what is wrong and why |
| **Threat** | Who or what would realistically exploit this? |
| **Vulnerability** | The specific weakness (misconfiguration, design flaw, missing control) |
| **Attack Scenario** | Step-by-step description of how exploitation occurs |
| **Evidence** | Screenshots, output, logs, or other artifacts proving the finding |
| **Impact** | What would a real adversary achieve if this were exploited? |
| **Likelihood** | Assessment of how likely exploitation is, accounting for exposure and existing controls |
| **Risk** | Combined assessment of impact and likelihood |
| **Existing Controls** | Controls currently in place and their effectiveness |
| **Control Failure** | Why existing controls did not prevent this finding |
| **Root Cause** | The underlying cause: design failure, configuration failure, process failure, or knowledge gap |
| **Recommendation** | What should be done to address the finding |
| **Remediation** | Specific technical or process change to implement |
| **Residual Risk** | Risk that remains after the recommended remediation is implemented |
| **Retest** | Criteria for confirming the finding is resolved |
| **References** | Relevant standards, CVEs, or external resources |

---

## Risk-Based Severity Rating

Use [[Risk-Based Offensive Security]] to determine severity — not CVSS base score alone.

| Severity | Meaning |
|---|---|
| **Critical** | High-likelihood path to significant business impact; existing controls are absent or ineffective; immediate action required |
| **High** | Realistic exploitation path; significant business impact; controls provide limited protection; action required within defined SLA |
| **Medium** | Exploitation possible but requires conditions; moderate impact; some controls effective; remediation planned |
| **Low** | Exploitation unlikely, impact limited, or compensating controls effective; remediation at next opportunity |
| **Informational** | Weakness observed with no direct exploitation path; or negligible impact; awareness only |

---

## Writing for Multiple Audiences

A professional finding should be understandable to technical, risk, and management audiences. This does not mean three separate documents — it means layering the content:

### Management Layer (Executive Summary)

One paragraph per finding maximum. Focus on: what was found, why it matters to the business, what the risk is, what needs to happen. No jargon.

**Poor:**
> "We observed that unconstrained Kerberos delegation was enabled on FILESERVER01 enabling S4U2self ticket forging via SPN coercion."

**Better:**
> "A configuration weakness on a file server could allow an attacker who has already compromised any domain user account to gain administrative control of that server. This creates a path to sensitive data stored on the server and provides a platform for further compromise of the environment."

---

### Risk Layer (Finding Detail)

Threat, vulnerability, attack scenario, impact, likelihood, risk. For risk professionals. Should connect to the organization's risk register.

---

### Technical Layer (Evidence and Remediation)

Evidence, reproduction steps, technical remediation, retest criteria. For the team implementing fixes.

---

## Report Structure

| Section | Content |
|---|---|
| **Cover** | Engagement details, scope, dates, classification, distribution |
| **Executive Summary** | 1-2 pages. Key findings by risk. Summary of most important actions. Overall security posture assessment. |
| **Scope and Methodology** | What was tested. What was not tested. Testing types used. Limitations. |
| **Findings Summary** | Table of all findings: ID, title, severity, affected asset, status |
| **Findings Detail** | Full finding template for each finding |
| **Appendices** | Scope list, tool list, methodology detail, glossary |

---

## Management Translation Examples

| Technical Finding | Management Translation |
|---|---|
| "Kerberoastable service account with Domain Admin group membership" | "A configuration that is standard in the environment allows any domain user to attempt to crack the password of an account that has full administrative control over the domain. If successful, an adversary would have complete control of all systems." |
| "LDAP signing not enforced — NTLM relay potential" | "A network configuration gap may allow an attacker on the internal network to intercept and redirect authentication to capture administrative credentials without the user's knowledge." |
| "Exposed S3 bucket with PII" | "Customer data was found accessible without authentication from the internet, creating regulatory exposure and breach notification risk." |

See: [[Security Validation - Management View]] for the broader management communication framework.

---

## Connecting to the Operating Model

Every finding should trace back to:

```text
Finding
  → Affected Asset (from asset inventory)
  → Business Risk (from [[02 - IT RISK MANAGEMENT]] or [[01 - BUSINESS & GOVERNANCE]])
  → Control Failure (from [[Security Controls - Index]])
  → Root Cause (design / configuration / process / knowledge)
  → Remediation (control improvement or control addition)
  → Residual Risk (input to management decision)
```

---

## Retest

Every finding requires a defined retest:

| Field | Content |
|---|---|
| Retest Condition | What change must be implemented before retest |
| Retest Criteria | How the tester will confirm the finding is resolved |
| Retest Result | Resolved / Partially Resolved / Not Resolved |
| Evidence | Confirmation screenshot, output, or configuration review |

---

## Related Notes

- [[Template - Professional Finding (Full)]] — the full finding template
- [[Risk-Based Offensive Security]] — how to determine severity
- [[Security Validation - Management View]] — management communication framework
- [[Offensive Security Methodology]] — Phase 14 (Reporting)
- [[Operation Cypher-Knife - Project Bridge]] — authorized operation findings and reporting
- [[VAPT - Project Bridge]] — assessment methodology reporting
