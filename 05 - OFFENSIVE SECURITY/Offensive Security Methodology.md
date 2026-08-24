---
title: Offensive Security Methodology
category: Methodology
tags:
  - OffensiveSecurity
  - Methodology
  - Engagement
date_created: 2026-08-24
status: MAINTAINED
---

# Offensive Security Methodology

## Executive Summary

A professional offensive security engagement follows a structured methodology to ensure that testing is authorized, evidence-based, business-relevant, and produces actionable findings. Every phase has a defined objective, inputs, activities, outputs, risks, and exit criteria.

**Critical Principle:** No phase begins before its authorization and exit criteria are confirmed. Do not start testing without written authorization and a defined scope.

---

## The 17-Phase Methodology

```text
1. Authorization
2. Scope
3. Rules of Engagement
4. Asset Discovery
5. Reconnaissance
6. Enumeration
7. Attack Surface Analysis
8. Vulnerability Identification
9. Risk Validation
10. Controlled Exploitation
11. Impact Assessment
12. Evidence Collection
13. Cleanup
14. Reporting
15. Remediation Support
16. Retesting
17. Lessons Learned
```

---

## Phase 1 — Authorization

| Field | Details |
|---|---|
| **Objective** | Obtain documented permission from the asset owner before any testing activity begins |
| **Inputs** | Engagement request, asset inventory, proposed scope, legal entity confirmation |
| **Activities** | Obtain written authorization signed by the appropriate asset owner; confirm legal jurisdiction; identify any regulated environments (healthcare, financial, critical infrastructure); confirm emergency stop contacts |
| **Outputs** | Signed authorization document, emergency contact list, engagement kickoff record |
| **Risks** | Testing without authorization is unauthorized access regardless of intent |
| **Evidence** | Signed authorization; dated kickoff record |
| **Exit Criteria** | Written authorization confirmed by asset owner before any technical activity |

> VERIFY: Authorization requirements vary by jurisdiction and regulatory environment. Do not assume a verbal agreement is sufficient.

---

## Phase 2 — Scope

| Field | Details |
|---|---|
| **Objective** | Define precisely what is in scope and what is explicitly excluded |
| **Inputs** | Authorization document, asset inventory, business context |
| **Activities** | Document in-scope systems, networks, applications, identities, and data; document explicit exclusions; document production system handling; identify third-party dependencies that require separate authorization |
| **Outputs** | Written scope statement with in-scope and out-of-scope lists |
| **Risks** | Ambiguous scope creates testing creep and unauthorized activity |
| **Evidence** | Scope document signed or acknowledged by client/owner |
| **Exit Criteria** | Scope document reviewed and confirmed before active testing begins |

---

## Phase 3 — Rules of Engagement

| Field | Details |
|---|---|
| **Objective** | Agree on operational constraints that govern how testing is conducted |
| **Inputs** | Authorization, scope, client operational requirements |
| **Activities** | Define permitted testing hours; confirm testing from approved IP addresses; agree on notification procedures for critical findings; define what constitutes an emergency stop; confirm data handling requirements; agree on communication channels |
| **Outputs** | Rules of engagement document |
| **Risks** | Undocumented rules create disputes; aggressive testing without agreed constraints can cause service disruptions |
| **Evidence** | Signed or acknowledged ROE document |
| **Exit Criteria** | ROE agreed before scanning or enumeration begins |

---

## Phase 4 — Asset Discovery

| Field | Details |
|---|---|
| **Objective** | Identify assets within scope that may not be fully documented in the client's inventory |
| **Inputs** | Scope, authorization, asset inventory provided by client |
| **Activities** | Confirm known assets; identify additional hosts, services, or applications within scope that may have been missed; validate that discovered assets are in scope before proceeding |
| **Outputs** | Validated asset list; delta between client-provided inventory and discovered assets |
| **Risks** | Testing an asset that is out of scope due to undocumented shared infrastructure |
| **Evidence** | Asset discovery log with timestamps and authorized IP sources |
| **Exit Criteria** | Confirmed asset inventory within scope |

---

## Phase 5 — Reconnaissance

| Field | Details |
|---|---|
| **Objective** | Gather information about the target from available sources to inform attack surface analysis |
| **Inputs** | Scope, asset list |
| **Activities** | Passive reconnaissance: DNS records, certificate transparency, public registrations, job postings, code repositories, social engineering indicators; active reconnaissance (where authorized): network scanning, service identification |
| **Outputs** | Reconnaissance notes: exposed domains, IP ranges, technologies, potential credential indicators, externally visible attack surface |
| **Risks** | Active reconnaissance may be detected; passive reconnaissance must not involve unauthorized system interaction |
| **Evidence** | Timestamped reconnaissance output referencing source |
| **Exit Criteria** | Sufficient information to proceed to enumeration; gaps noted |

See also: [[Reconnaissance - Concepts]]

---

## Phase 6 — Enumeration

| Field | Details |
|---|---|
| **Objective** | Identify specific services, identities, permissions, and configurations within the authorized scope |
| **Inputs** | Reconnaissance output, asset list |
| **Activities** | Service and version enumeration; identity and account enumeration; permission and share enumeration; configuration and misconfiguration identification; Active Directory enumeration (where in scope) |
| **Outputs** | Enumeration findings: services, accounts, permissions, configuration weaknesses |
| **Risks** | Enumeration may generate alerts; document all enumeration sources and timestamps |
| **Evidence** | Enumeration output with timestamps |
| **Exit Criteria** | Services, identities, and configuration landscape understood for the in-scope environment |

See also: [[Enumeration - Concepts]]

---

## Phase 7 — Attack Surface Analysis

| Field | Details |
|---|---|
| **Objective** | Analyse what has been discovered and identify where meaningful security weaknesses exist |
| **Inputs** | Reconnaissance and enumeration output |
| **Activities** | Map attack surface: external exposure, authentication weaknesses, configuration gaps, identity risks, patch status, network segmentation; prioritise areas of highest potential business risk for further investigation |
| **Outputs** | Attack surface map; prioritised testing targets |
| **Risks** | Focus on technically interesting areas rather than business-relevant ones; misalignment with scope |
| **Evidence** | Attack surface analysis document |
| **Exit Criteria** | Testing priorities agreed with engagement lead before proceeding to exploitation |

---

## Phase 8 — Vulnerability Identification

| Field | Details |
|---|---|
| **Objective** | Identify specific vulnerabilities in prioritized areas of the attack surface |
| **Inputs** | Attack surface analysis |
| **Activities** | Vulnerability scanning (automated); manual validation of scan findings; manual vulnerability identification not detected by scanners; configuration review; authentication testing; authorization testing |
| **Outputs** | Validated vulnerability list with: affected asset, vulnerability description, confidence level |
| **Risks** | Automated scanning alone misses logic and design flaws; over-reliance on CVSS without business context |
| **Evidence** | Scanner output; manual validation notes; screenshot or log evidence |
| **Exit Criteria** | Vulnerability list reviewed and prioritized before controlled exploitation |

See also: [[Vulnerability Assessment - Concepts]]

---

## Phase 9 — Risk Validation

| Field | Details |
|---|---|
| **Objective** | Assess whether identified vulnerabilities represent real business risk before attempting exploitation |
| **Inputs** | Vulnerability list, business context, asset criticality, existing control information |
| **Activities** | Evaluate: Is the asset critical? Is the vulnerability exploitable in this environment? What existing controls may reduce likelihood or impact? What would the realistic business impact be? Is exploitation proportionate and authorized? |
| **Outputs** | Prioritized exploitation plan with risk justification |
| **Risks** | Wasting effort on technically interesting but low-business-risk vulnerabilities; missing simple, high-risk misconfigurations |
| **Evidence** | Risk validation notes referencing asset criticality and existing controls |
| **Exit Criteria** | Exploitation plan reviewed and authorized before proceeding |

See also: [[Risk-Based Offensive Security]]

---

## Phase 10 — Controlled Exploitation

| Field | Details |
|---|---|
| **Objective** | Demonstrate exploitability under controlled, authorized conditions to produce evidence for findings |
| **Inputs** | Authorized exploitation plan, ROE, emergency contacts |
| **Activities** | Execute authorized exploitation techniques; document every action with timestamp; remain within defined scope; stop immediately if unexpected impact is observed; report critical findings immediately per agreed notification procedure |
| **Outputs** | Exploitation evidence: screenshots, command output, log captures; access level achieved; impact observed |
| **Risks** | Unexpected service disruption; lateral scope creep; out-of-scope system interaction |
| **Evidence** | Timestamped exploitation log; screenshots; captured output |
| **Exit Criteria** | Exploitation objective achieved OR scope limit reached OR stop condition triggered |

> **LAB vs AUTHORIZED ASSESSMENT vs PRODUCTION**
> - LAB: Isolated environment under your control. No authorization required beyond lab ownership.
> - AUTHORIZED ASSESSMENT: Live environment with written authorization. Every action is logged. Emergency contacts are active.
> - PRODUCTION: Heightened care. Confirm ROE explicitly addresses production risk. Some techniques are prohibited in production.

---

## Phase 11 — Impact Assessment

| Field | Details |
|---|---|
| **Objective** | Determine what a real adversary could have achieved if exploitation had continued beyond the point demonstrated |
| **Inputs** | Exploitation results, asset criticality, business context |
| **Activities** | Assess: What data was accessible? What systems were reachable? What business process could have been disrupted? What is the realistic impact if a real adversary used this path? |
| **Outputs** | Impact statement for each finding |
| **Risks** | Overstating impact (creates alarm without evidence); understating impact (finding is deprioritized incorrectly) |
| **Evidence** | Impact evidence tied to exploitation results; asset criticality documentation |
| **Exit Criteria** | Impact statement written before findings are drafted |

---

## Phase 12 — Evidence Collection

| Field | Details |
|---|---|
| **Objective** | Compile all evidence required to support findings, remediation, and retest |
| **Inputs** | All phase outputs |
| **Activities** | Organize evidence by finding; confirm each finding has: affected asset, description, reproduction steps, evidence, impact statement; hash or timestamp evidence to demonstrate integrity |
| **Outputs** | Evidence package per finding |
| **Risks** | Incomplete evidence undermines finding credibility; sensitive evidence must be handled per agreed data handling requirements |
| **Evidence** | Evidence files; chain of custody if forensic-grade handling is required |
| **Exit Criteria** | Every finding has sufficient evidence for remediation and retest |

---

## Phase 13 — Cleanup

| Field | Details |
|---|---|
| **Objective** | Remove any artifacts, accounts, tools, or persistence left during testing |
| **Inputs** | Log of all actions taken during testing |
| **Activities** | Remove test accounts; delete tools, payloads, and staged files; remove persistence mechanisms; restore modified configurations; document cleanup actions; confirm with client |
| **Outputs** | Cleanup confirmation log |
| **Risks** | Leftover artifacts create operational risk; failure to clean up may be discovered during production incidents |
| **Evidence** | Cleanup log with timestamps |
| **Exit Criteria** | All artifacts removed and confirmed with client before report delivery |

---

## Phase 14 — Reporting

| Field | Details |
|---|---|
| **Objective** | Produce a professional report that communicates findings to technical, risk, and management audiences |
| **Inputs** | All findings, evidence packages, impact assessments |
| **Activities** | Draft executive summary; draft each finding using the professional finding template; organize findings by risk; include methodology summary; include remediation recommendations; include retest plan |
| **Outputs** | Draft report; final report after client review |
| **Risks** | Findings without business context are not actionable for management; overly technical reports exclude risk owners |
| **Evidence** | Report document |
| **Exit Criteria** | Report reviewed by engagement lead; findings confirmed accurate before delivery |

See also: [[Offensive Security Reporting]]

---

## Phase 15 — Remediation Support

| Field | Details |
|---|---|
| **Objective** | Support the client in understanding and implementing remediations |
| **Inputs** | Delivered report |
| **Activities** | Clarify findings on request; explain remediation recommendations; assist in understanding risk context; do not implement remediations in client environments without separate authorization |
| **Outputs** | Remediation clarifications; updated finding status |
| **Risks** | Providing unclear guidance leads to ineffective remediation and failed retest |
| **Evidence** | Communication record |
| **Exit Criteria** | Remediations confirmed implemented; retest scheduled |

---

## Phase 16 — Retesting

| Field | Details |
|---|---|
| **Objective** | Confirm that remediations have effectively resolved identified findings |
| **Inputs** | Remediated findings; original evidence |
| **Activities** | Reproduce original exploitation conditions; confirm finding is resolved OR confirm partial resolution with remaining risk |
| **Outputs** | Retest result per finding: Resolved / Partially Resolved / Not Resolved |
| **Risks** | Incomplete remediation creates false confidence; partial fixes may introduce new issues |
| **Evidence** | Retest evidence: confirmation of fix or remaining vulnerability |
| **Exit Criteria** | All findings retested; retest report issued |

---

## Phase 17 — Lessons Learned

| Field | Details |
|---|---|
| **Objective** | Extract value from the engagement beyond individual findings |
| **Inputs** | Full engagement output; retest results |
| **Activities** | Identify systemic weaknesses (not just individual findings); identify detection gaps; identify root causes; feed into risk register; feed into security awareness; feed into detection engineering; update internal methodology if improved |
| **Outputs** | Lessons learned record; risk register updates; detection improvement referrals |
| **Risks** | Lessons learned step skipped → patterns repeat |
| **Evidence** | Lessons learned note |
| **Exit Criteria** | Lessons learned documented and routed to appropriate owners |

See also: [[Purple Team - Lessons Learned Model]]

---

## Related Notes

- [[Security Testing Types]] — understand what type of engagement you are running
- [[Offensive Security Safety and Authorization]] — authorization requirements in detail
- [[Risk-Based Offensive Security]] — how to evaluate finding priority
- [[Offensive Security Reporting]] — finding template and reporting guidance
- [[Control Testing Methodology]] — how control testing connects to offensive validation
- [[VAPT - Project Bridge]] — assessment methodology project
- [[GOAD Project - Risk and Control Bridge]] — Active Directory laboratory
- [[Operation Cypher-Knife - Project Bridge]] — authorized operation case study
