---
title: Risk-Based Offensive Security
category: Methodology
tags:
  - OffensiveSecurity
  - Risk
  - BusinessContext
  - Findings
date_created: 2026-08-24
status: MAINTAINED
---

# Risk-Based Offensive Security

## The Core Principle

> **A technically impressive vulnerability is not automatically a high business risk.**

The goal of professional offensive security is to identify weaknesses that create meaningful risk to the organization — not to demonstrate that a technique works. A technique that succeeds in a lab but is already detected, already compensated for, and affects only a non-critical asset may be a low priority. A simple misconfiguration that allows lateral movement to a Crown Jewels system is a critical finding regardless of technical complexity.

---

## Why This Matters

### Common Failure Mode: Overstated Technical Findings

A tester demonstrates successful exploitation of a vulnerability that:
- Affects a decommissioned system
- Requires physical access
- Is already detected by EDR
- Has no path to sensitive data
- Has a compensating control that fully mitigates the realistic impact

The finding is technically correct — the vulnerability exists and was exploited. But it is **not a high business risk**.

### Common Failure Mode: Understated Business-Relevant Findings

A tester identifies:
- A service account with excessive permissions used by an application
- Domain-joined workstations that can query all user attributes including sensitive HR data
- A cloud storage bucket with access logs disabled

These may not trigger automated scanner findings. They may not have a CVE. But they create significant business risk.

---

## Risk Factors for Offensive Findings

Every finding should be assessed across these dimensions before assigning severity:

### Asset Criticality

| Question | Why It Matters |
|---|---|
| What is this asset? | Systems and data have different criticality to the business |
| What business process does it support? | Compromise of a system supporting revenue-critical processes is higher impact |
| Is the asset classified? | Classification (e.g., Confidential, Regulated) affects impact assessment |
| Is the asset in scope of regulatory obligations? | Regulatory environments VERIFY — jurisdiction-specific |
| Who owns the asset? | Ownership determines who must make the risk decision |

### Exposure

| Question | Why It Matters |
|---|---|
| Is the vulnerability externally accessible? | Internet-exposed weaknesses have higher likelihood |
| Is the vulnerable service exposed internally? | Internal exposure with broad network access is still significant |
| How many actors could reach this? | Broader exposure = higher likelihood |
| Is access authenticated or unauthenticated? | Unauthenticated exposure is nearly always more severe |

### Exploitability

| Question | Why It Matters |
|---|---|
| Is exploitation reliable and repeatable? | Unreliable exploitation may not represent realistic adversary capability |
| Is a public exploit available? | Lowers the skill threshold required |
| Does exploitation require chaining multiple weaknesses? | Complex chains are lower likelihood but still valid if the impact is high |
| What level of access is required to begin exploitation? | Unauthenticated > authenticated-user > administrator-required |

### Threat

| Question | Why It Matters |
|---|---|
| Who would realistically exploit this? | External threat actor, insider, compromised vendor |
| Is this organization a realistic target for sophisticated actors? | Threat relevance affects likelihood |
| Is this technique actively used in the wild? | Threat intelligence can inform likelihood |

See: [[Threat Intelligence - Overview]]

### Business Impact

| Question | Why It Matters |
|---|---|
| What data could be accessed? | Data sensitivity directly affects impact |
| What systems could be reached from here? | Lateral movement potential multiplies impact |
| What business process could be disrupted? | Availability impact must be assessed |
| What is the regulatory or legal impact of this? | VERIFY per jurisdiction |
| What is the reputational impact? | Hard to quantify but real |

### Existing Controls

| Question | Why It Matters |
|---|---|
| Is there a preventive control that reduces likelihood? | Even an imperfect control reduces risk |
| Is there a detective control that would identify exploitation? | Detection reduces dwell time and impact |
| Is there a corrective control that would reduce impact? | Recovery capability matters |
| Are the controls operating effectively? | A control that exists but is not operating does not reduce risk |
| Are there compensating controls? | Compensating controls can materially change the risk rating |

See: [[Security Controls - Index]], [[Control Effectiveness Model]]

### Likelihood

Combine: Exposure + Exploitability + Threat → realistic likelihood that this finding would be exploited.

> Do not use CVSS base scores as a proxy for business risk. CVSS describes the technical characteristics of a vulnerability — it does not account for your specific asset criticality, your specific controls, or your specific threat environment.

### Residual Risk

After accounting for all existing controls and compensating controls — what risk remains?

This is the residual risk that management must decide to accept, mitigate further, transfer, or avoid.

---

## The Risk Assessment Table

Apply this table for each finding:

| Factor | Assessment | Notes |
|---|---|---|
| Asset Criticality | Critical / High / Medium / Low | Based on business process and data |
| Exposure | Internet / Internal-Broad / Internal-Restricted / Local-Only | |
| Exploitability | Reliable / Probable / Complex / Theoretical | |
| Threat | High-Relevance / Moderate / Low | Based on threat intelligence or context |
| Business Impact | Critical / Significant / Moderate / Minimal | |
| Existing Controls | None / Compensating / Partial / Effective | |
| Detection Capability | None / Partial / Effective | From [[Detection Validation - Overview]] |
| Likelihood | High / Medium / Low | Synthesis of above |
| Residual Risk | Critical / High / Medium / Low | After controls |

---

## Risk-Based Severity Ratings

| Rating | Meaning |
|---|---|
| **Critical** | High-likelihood path to high-impact compromise of critical assets; existing controls ineffective or absent |
| **High** | Realistic exploitation path; significant business impact; controls provide limited protection |
| **Medium** | Exploitation possible but requires conditions; moderate impact; some controls effective |
| **Low** | Exploitation unlikely, impact limited, or effective controls reduce realistic risk |
| **Informational** | Weakness observed; no direct exploitation path; or impact is negligible relative to compensating controls |

> Do not assign Critical severity because a technique is technically sophisticated. Assign it because the combination of likelihood and business impact, accounting for existing controls, creates a risk the organization needs to address urgently.

---

## Connecting Findings to Batch 1 and Batch 2

Every finding should trace back through the operating model:

```text
Finding (technical)
  → Affected Asset (from asset inventory or business context)
  → Business Risk (from [[02 - IT RISK MANAGEMENT]] or [[01 - BUSINESS & GOVERNANCE]])
  → Control Failure (from [[Security Controls - Index]])
  → Root Cause (design failure, configuration failure, operational failure)
  → Remediation (control improvement, configuration change, process change)
  → Residual Risk (management decision input)
```

---

## Related Notes

- [[Cybersecurity Operating Model]] — full chain from business risk to management decision
- [[Offensive Security Reporting]] — how to write risk-based findings
- [[Control Failure Workflow]] — what to do when a control fails
- [[Control Effectiveness Model]] — how to assess control quality
- [[Security Validation - Management View]] — how to translate findings for management
- [[Template - Professional Finding (Full)]] — finding template with all risk fields
