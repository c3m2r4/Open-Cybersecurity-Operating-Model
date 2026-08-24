---
title: Control Validation - Maturity Model
category: Methodology
tags:
  - Validation
  - Maturity
  - PurpleTeam
date_created: 2026-08-24
status: MAINTAINED
---

# Control Validation — Maturity Model

## Purpose

An organization does not implement full continuous automated validation on day one. A maturity model provides a roadmap for evolving the validation capability from ad-hoc compliance checks to threat-informed, continuous assurance.

> Attempting to implement continuous automated validation before basic telemetry and SIEM capabilities exist will result in massive noise and failed deployments. Follow the progression.

---

## The Maturity Levels

### Level 1: Ad-Hoc / Compliance Driven
The organization tests controls primarily to satisfy auditors or external requirements, not to validate threat mitigation.

| Characteristic | Detail |
|---|---|
| **Driver** | Compliance, PCI, SOC2, annual audit |
| **Frequency** | Annual (Penetration Test) |
| **Scope** | Limited to compliant networks or specific applications |
| **Remediation** | Fix the specific vulnerability found to pass the audit |
| **Metrics** | "Did we pass the pentest?" |

### Level 2: Scheduled Vulnerability Management
The organization proactively identifies vulnerabilities but focuses on patching known CVEs rather than validating behavioral controls.

| Characteristic | Detail |
|---|---|
| **Driver** | IT hygiene and vulnerability reduction |
| **Frequency** | Weekly/Monthly (Vulnerability Scanning) |
| **Scope** | Broad internal and external networks |
| **Remediation** | Patching systems and updating software |
| **Metrics** | Scan coverage; critical vulnerabilities patched within SLA |

### Level 3: Targeted Validation (Purple Teaming)
The organization begins validating how controls respond to specific attacker behaviors (TTPs), not just missing patches.

| Characteristic | Detail |
|---|---|
| **Driver** | Threat intelligence and operational improvement |
| **Frequency** | Quarterly (Purple Team Exercises) |
| **Scope** | Critical controls (EDR, SIEM, Identity) and high-value assets |
| **Remediation** | Tuning detection rules, adjusting control configurations |
| **Metrics** | TTP detection coverage; time to tune (improvement over baseline) |

### Level 4: Continuous Automated Validation
The organization automates the execution of simulated attacks to ensure controls do not drift over time.

| Characteristic | Detail |
|---|---|
| **Driver** | Preventing configuration drift and ensuring continuous assurance |
| **Frequency** | Daily / Continuous |
| **Scope** | Enterprise-wide automated deployment (BAS tools) |
| **Remediation** | Automated alerting to SOC/Engineering when a control fails |
| **Metrics** | Continuous control effectiveness percentage; drift detection rate |

### Level 5: Threat-Informed Defense
The organization dynamically aligns its validation program with active threat intelligence, shifting focus as the threat landscape changes.

| Characteristic | Detail |
|---|---|
| **Driver** | Current, actionable threat intelligence |
| **Frequency** | Continuous + Ad-hoc based on emerging intel |
| **Scope** | Full enterprise, aligned to threat actor profiles |
| **Remediation** | Pre-emptive tuning against emerging threats before they attack |
| **Metrics** | Resilience against targeted threat actor profiles |

---

## Progression Milestones

How to move from one level to the next:

| Transition | Required Capability |
|---|---|
| **Level 1 → Level 2** | Implementation of enterprise vulnerability scanning and a patching process SLA. |
| **Level 2 → Level 3** | Establishment of a SIEM with telemetry, and execution of the first collaborative Purple Team exercise. |
| **Level 3 → Level 4** | Procurement and deployment of Breach and Attack Simulation (BAS) tools and integration into CI/CD for detection rules. |
| **Level 4 → Level 5** | Mature Cyber Threat Intelligence (CTI) capability that feeds directly into the validation planning process. |

---

## Related Notes

- [[Control Validation - Framework]] — the methodology used at Level 3 and above
- [[Purple Team - Lifecycle]] — the core activity of Level 3
- [[Threat Intelligence - Overview]] — the requirement for Level 5
