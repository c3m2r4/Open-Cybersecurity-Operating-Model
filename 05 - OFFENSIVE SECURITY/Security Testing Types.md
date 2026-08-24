---
title: Security Testing Types
category: Methodology
tags:
  - OffensiveSecurity
  - Taxonomy
  - SecurityTesting
date_created: 2026-08-24
status: MAINTAINED
---

# Security Testing Types

## Why This Matters

These terms are used interchangeably in the industry. Using them imprecisely creates misaligned expectations between clients, testers, and management. A penetration test is not a red team engagement. A vulnerability scan is not a penetration test.

This note defines each type clearly so that scope, authorization, and expected output are understood before work begins.

---

## The Testing Spectrum

```text
Vulnerability Scanning
    ↓
Vulnerability Assessment
    ↓
Penetration Testing
    ↓
Red Teaming
    ↓
Adversary Simulation
    ↑
Purple Teaming (different axis — collaborative)
    ↑
Security Control Validation (discrete, targeted)
```

---

## Type Definitions

### Vulnerability Scanning

| Field | Detail |
|---|---|
| **Definition** | Automated tool-based identification of known vulnerabilities in systems, services, or applications |
| **Purpose** | Detect known weaknesses at scale |
| **Scope** | Broad — typically all systems within an IP range or application inventory |
| **Authorization** | Required; production scanning must be coordinated with operations |
| **Operator** | Can be run by operations, engineering, or security teams |
| **Output** | Vulnerability list with severity ratings (typically CVSS-based) |
| **Limitations** | Does not validate exploitability; produces false positives; misses logic flaws and design weaknesses; misses misconfiguration that is not a known CVE |
| **Business Value** | Broad detection of patch and configuration gaps at low cost |

> A vulnerability scanner finds what the scanner knows about. It does not tell you whether the vulnerability is actually exploitable in your environment, or whether your controls would detect or prevent exploitation.

---

### Vulnerability Assessment

| Field | Detail |
|---|---|
| **Definition** | A structured process that combines scanning with manual analysis to confirm, prioritize, and contextualize vulnerabilities |
| **Purpose** | Produce a validated, risk-contextualized view of weaknesses in a defined scope |
| **Scope** | Defined — specific systems, applications, or environments |
| **Authorization** | Required |
| **Operator** | Security specialist or analyst |
| **Output** | Validated vulnerability list with business context, risk prioritization, and remediation guidance |
| **Limitations** | Validates that a vulnerability exists; does not typically demonstrate full exploitation or business impact |
| **Business Value** | More reliable than raw scan output; prioritized remediation roadmap |

> A vulnerability assessment tells you what is weak and why it matters. It may not tell you whether an attacker could actually exploit the weakness end-to-end in your environment.

---

### Penetration Testing

| Field | Detail |
|---|---|
| **Definition** | An authorized, structured attempt to exploit identified vulnerabilities to demonstrate real-world exploitability and business impact |
| **Purpose** | Prove that weaknesses can be chained into real attack paths; identify the actual impact of compromise |
| **Scope** | Defined and agreed — specific systems, applications, network segments, or identity environment |
| **Authorization** | Written authorization required from asset owner |
| **Operator** | Qualified security specialist with offensive security skills |
| **Output** | Professional report with validated findings, exploitation evidence, impact assessment, and remediation guidance |
| **Limitations** | Point-in-time; bounded by agreed scope; does not test organizational detection and response capability; does not simulate full adversary campaign |
| **Business Value** | Evidence-based confirmation of exploitability; business impact assessment; prioritized remediation |

> A penetration test tells you whether an attacker could succeed against the specific vulnerabilities and configurations you tested. It does not tell you whether your detection and response capability would identify a real attack.

---

### Red Teaming

| Field | Detail |
|---|---|
| **Definition** | An objective-based adversary emulation engagement that tests the organization's detection, response, and resilience without disclosing the test to the security team |
| **Purpose** | Test the organization's ability to detect, investigate, and respond to a realistic adversary campaign |
| **Scope** | Broad — typically the full organization or a significant business unit; specific objectives are defined (e.g., reach Crown Jewels) |
| **Authorization** | Written authorization from senior leadership; typically unknown to the SOC and security operations team |
| **Operator** | Specialized red team practitioners |
| **Output** | Campaign narrative; findings; detection gaps; response gaps; management brief |
| **Limitations** | Resource-intensive; not suited for organizations without a functioning detection and response capability; findings may be difficult to act on without defensive maturity |
| **Business Value** | Real-world test of organizational resilience; exposes blind spots in detection and response |

> A red team engagement tests whether your security operations team would detect and respond to a real adversary, not just whether a vulnerability exists.

---

### Adversary Simulation

| Field | Detail |
|---|---|
| **Definition** | A red team engagement that emulates the specific tactics, techniques, and procedures (TTPs) of a named threat actor relevant to the organization |
| **Purpose** | Test organizational resilience against threat actors that are likely to target the organization based on its sector, geography, or profile |
| **Scope** | Broad, scenario-driven |
| **Authorization** | Written authorization from senior leadership |
| **Operator** | Specialized practitioners with threat intelligence capability |
| **Output** | Campaign findings mapped to threat actor TTPs; organizational resilience assessment against specific threat |
| **Limitations** | Requires high-quality threat intelligence; results are only meaningful if the threat actor is genuinely relevant |
| **Business Value** | Directly connects security investment to relevant adversary capability |

See also: [[Threat Intelligence - Overview]], [[MITRE ATT&CK - Overview]]

---

### Purple Teaming

| Field | Detail |
|---|---|
| **Definition** | A collaborative process in which offensive and defensive teams work together, in real time, to test attack techniques and validate detection and response capability |
| **Purpose** | Improve detection coverage, response playbooks, and control effectiveness through iterative, transparent collaboration |
| **Scope** | Specific attack techniques or attack scenarios; transparent to the defensive team |
| **Authorization** | Written authorization; defensive team is aware and participating |
| **Operator** | Offensive and defensive practitioners working jointly |
| **Output** | Detection and response improvement; control gap identification; remediation and retest results |
| **Limitations** | Does not test whether an unaware defensive team would detect an attack; requires both offensive and defensive capability |
| **Business Value** | Fastest path to measurable detection and response improvement; connects security investment to demonstrated capability |

> Purple teaming is not "red team + blue team in the same room." It is a structured, iterative process specifically designed to improve defensive capability.

See: [[07 - PURPLE TEAM/Purple Team - Index]]

---

### Security Control Validation

| Field | Detail |
|---|---|
| **Definition** | Targeted, specific testing that asks: "Does this control work as expected against this attack?" |
| **Purpose** | Confirm that a specific control is operating effectively against a specific threat scenario |
| **Scope** | Narrow — specific control, specific technique, specific environment |
| **Authorization** | Required |
| **Operator** | Can be security team, operations, or specialized practitioner |
| **Output** | Control result: Effective / Partially Effective / Ineffective; evidence |
| **Limitations** | Tests one control against one scenario — does not represent full attack-path coverage |
| **Business Value** | Directly validates that specific security investments are delivering expected protection |

See: [[Control Validation - Framework]], [[Control Testing Methodology]]

---

## Comparison Table

| Type | Scope | Defensive Team Awareness | Output | Primary Question |
|---|---|---|---|---|
| Vulnerability Scanning | Broad | N/A | Vulnerability list | What weaknesses exist? |
| Vulnerability Assessment | Defined | N/A | Validated vulnerability list + risk | Which weaknesses matter? |
| Penetration Testing | Defined | Typically not | Exploitation evidence + findings | Can it be exploited? What is the impact? |
| Red Teaming | Broad | No | Campaign narrative + gaps | Would we detect and respond? |
| Adversary Simulation | Broad | No | TTP-mapped campaign findings | Are we resilient against specific adversaries? |
| Purple Teaming | Technique-specific | Yes (collaborative) | Detection and response improvements | How do we improve capability now? |
| Control Validation | Narrow | N/A | Control result + evidence | Does this specific control work? |

---

## Choosing the Right Type

| Situation | Recommended Type |
|---|---|
| Need to find known patch gaps across all systems | Vulnerability Scanning |
| Need prioritized risk view of a specific application | Vulnerability Assessment |
| Need to know if a critical system could be compromised | Penetration Testing |
| Need to test whether the SOC would detect an attack | Red Teaming |
| Need to test against a specific known threat actor | Adversary Simulation |
| Need to rapidly improve detection coverage | Purple Teaming |
| Need to confirm a specific control works | Control Validation |

---

## Related Notes

- [[Offensive Security Methodology]] — engagement phases apply differently to each type
- [[Offensive Security Safety and Authorization]] — authorization requirements vary by type
- [[Purple Team - Lifecycle]] — purple team process detail
- [[Control Validation - Framework]] — control validation detail
