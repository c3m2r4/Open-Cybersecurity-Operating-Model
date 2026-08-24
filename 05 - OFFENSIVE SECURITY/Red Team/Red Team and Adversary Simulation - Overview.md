---
title: Red Team and Adversary Simulation - Overview
category: Security Domain
tags:
  - RedTeam
  - AdversarySimulation
  - OffensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Red Team and Adversary Simulation — Overview

## What Red Teaming Is (and Is Not)

| What Red Teaming IS | What Red Teaming IS NOT |
|---|---|
| Objective-based adversary emulation | A long penetration test |
| A test of detection and response capability | A vulnerability assessment |
| A campaign with a defined objective (reach Crown Jewels) | A demonstration of technique collection |
| Unknown to the security operations team | Collaborative with the defensive team (that is purple teaming) |
| Threat-informed | Framework box-checking |

The question a red team engagement answers: **"Would our security operations team detect, investigate, and respond to a real adversary campaign before the adversary achieved their objective?"**

See [[Security Testing Types]] for how this compares to other testing types.

---

## Red Team vs Purple Team

| Dimension | Red Team | Purple Team |
|---|---|---|
| SOC awareness | No | Yes — collaborative |
| Objective | Test detection/response | Improve detection/response |
| Speed of feedback | After the engagement | In real time |
| Best used when | Mature detection capability exists | Building or improving detection |
| Output | Campaign findings; detection/response gaps | Detection improvements; control improvements |

If the defensive team does not have a mature detection capability, a red team engagement will only find gaps without providing the iterative improvement mechanism. Purple team exercises are often more valuable at earlier maturity levels.

See [[07 - PURPLE TEAM/Purple Team - Index]].

---

## Campaign Structure

A red team engagement is a campaign, not a series of disconnected techniques:

```text
1. Define Objective
   (e.g., Obtain access to Crown Jewels system X without detection)
     ↓
2. Threat Intelligence
   (What TTPs would a relevant adversary use?)
     ↓
3. Reconnaissance
   (What is the external attack surface?)
     ↓
4. Initial Access
   (Phishing, credential stuffing, external service exploitation)
     ↓
5. Establish Foothold
   (C2 callback, persistence, avoid detection)
     ↓
6. Internal Reconnaissance
   (Understand environment; identify path to objective)
     ↓
7. Privilege Escalation / Lateral Movement
   (Move toward objective)
     ↓
8. Objective Achievement
   (Reach, access, or simulate impact on target)
     ↓
9. Cleanup and Documentation
     ↓
10. Report and Debrief
```

---

## Adversary Simulation

Adversary simulation extends red teaming by emulating the specific TTPs of a named threat actor:

| Concept | Detail |
|---|---|
| **Threat Actor Selection** | Based on threat intelligence relevant to the organization's sector, geography, and profile |
| **TTP Mapping** | Attack techniques mapped to the threat actor's known behavior (e.g., via MITRE ATT&CK) |
| **Emulation Fidelity** | Using the actor's actual tools, techniques, and operational patterns where possible |
| **Objective** | Test whether the organization can defend against that specific adversary |

> Adversary simulation is only meaningful if the threat actor being emulated is genuinely relevant to the organization. Simulating a nation-state APT when the organization's realistic threat is opportunistic ransomware is not an effective use of resources.

See: [[Threat Intelligence - Overview]], [[MITRE ATT&CK - Overview]]

---

## Operation Cypher-Knife Integration

Operation Cypher-Knife is the authorized operation case-study framework in this vault. It provides the framework for documenting objectives, scope, evidence, findings, and lessons learned for a red team or adversary simulation campaign.

| Cypher-Knife Area | Connection |
|---|---|
| [[Operation Cypher-Knife - Project Bridge]] | Project bridge linking the operation to risk and controls |
| [[Operation Cypher-Knife - Scope]] | Scope definition for the authorized operation |
| Objectives | Defined threat objective for the campaign |
| Evidence | Evidence collected during the operation |
| Findings | Risk-based findings from the campaign |
| Lessons Learned | Post-operation learning |

**Do not duplicate Operation Cypher-Knife procedures here.** Use this note for concepts; use Cypher-Knife notes for the operational record.

---

## Reporting a Red Team Engagement

Red team reports differ from penetration test reports:

| Penetration Test Report | Red Team Report |
|---|---|
| Finding-by-finding technical list | Campaign narrative with objective context |
| Vulnerability details | Attack path and dwell time |
| Remediation per finding | Detection and response gaps |
| Retest plan | Detection improvement plan |

A red team report should answer:
1. What was the objective?
2. Was the objective achieved?
3. Were we detected?
4. How long did we operate without detection?
5. What detection and response gaps were identified?
6. What needs to improve?

---

## Management Communication

A red team engagement produces powerful management communication if framed correctly:

| Bad Framing | Better Framing |
|---|---|
| "We successfully executed T1003.001 and obtained credential material." | "We operated undetected in the environment for 14 days and obtained access to the target system without triggering any security alerts." |
| "EDR did not detect our payload." | "The endpoint detection control did not identify the threat until after the objective was achieved, which would provide an adversary with sufficient time to exfiltrate sensitive data." |

See: [[Security Validation - Management View]]

---

## Related Notes

- [[Security Testing Types]] — red team vs other testing types
- [[Purple Team - Lifecycle]] — collaborative improvement alternative
- [[Threat Intelligence - Overview]] — threat intelligence informs adversary selection
- [[MITRE ATT&CK - Overview]] — TTP framework for adversary emulation
- [[Offensive Security Safety and Authorization]] — authorization requirements for red team
- [[Operation Cypher-Knife - Project Bridge]] — authorized operation case study
