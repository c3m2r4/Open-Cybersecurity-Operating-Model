---
title: SOC - Operating Model
category: Security Operations
tags:
  - SOC
  - SecurityOperations
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# SOC — Operating Model

## Executive Summary

A Security Operations Centre (SOC) is the team and capability responsible for monitoring, detecting, investigating, and responding to security events. Effective SOC operation depends on four equal pillars: People, Process, Technology, and Governance. A SOC that is strong in technology but weak in process will produce volume without outcomes. A SOC with strong process but insufficient technology will be unable to operate at scale.

---

## People

| Role | Primary Responsibility |
|---|---|
| **SOC Analyst (Tier 1)** | Alert triage; initial investigation; escalation of true positives |
| **SOC Analyst (Tier 2)** | Deeper investigation; threat hunting; case ownership |
| **SOC Analyst (Tier 3) / IR Lead** | Incident response; forensic investigation; threat hunting; escalation |
| **Detection Engineer** | Build, test, tune, and maintain detection rules; manage SIEM use cases |
| **Threat Intelligence Analyst** | Produce and consume intelligence to inform detection and hunting priorities |
| **SOC Manager** | People management; escalation; stakeholder reporting; process ownership |
| **Incident Response Lead** | Owns IR during declared incidents; coordinates with legal, comms, and management |

> Staffing models vary significantly by organization size and maturity. VERIFY role definitions against your organization's specific structure.

### Analyst Competencies

| Level | Expected Competency |
|---|---|
| Tier 1 | Alert triage; tool proficiency; escalation criteria; SIEM queries; playbook execution |
| Tier 2 | Investigation methodology; log analysis; hypothesis formation; indicator analysis; case writing |
| Tier 3 | Forensic analysis; threat hunting; adversary behavior; detection logic; IR management |

---

## Process

### Alert Lifecycle

```text
Alert Generated
  ↓
Triage (Is this alert worth investigating?)
  ↓
Investigation (What happened? What is the scope?)
  ↓
Classification (True Positive / False Positive / Benign True Positive)
  ↓
[If True Positive]
  ↓
Containment (Stop the spread)
  ↓
Eradication (Remove the threat)
  ↓
Recovery (Restore normal operations)
  ↓
Lessons Learned (Improve detections, playbooks, controls)
```

### Triage Standard

| Step | Action |
|---|---|
| **Initial assessment** | Is this alert high-fidelity? Is this a known false positive pattern? |
| **Context gathering** | What is the affected asset? What is its criticality? What user or process is involved? |
| **Enrichment** | Enrich indicators: IP reputation, domain age, file hash, user account history |
| **Decision** | Escalate / close as false positive / close as benign true positive |

### Investigation Standard

| Question | Answer Source |
|---|---|
| What happened? | Alert detail; related log events; timeline |
| What is affected? | Asset inventory; endpoint telemetry; identity logs |
| How long was this occurring? | Log retention; first-seen indicator; authentication history |
| Is this isolated or part of a wider campaign? | Pivot on indicators; lateral movement evidence; related alerts |
| What is the severity? | Impact on asset criticality; data accessed; systems affected |

### Classification

| Classification | Meaning |
|---|---|
| **True Positive** | Alert is confirmed as malicious activity; incident response begins |
| **False Positive** | Alert triggered on benign activity; detection requires tuning |
| **Benign True Positive** | Alert is technically correct but the activity is authorized or expected |
| **Inconclusive** | Insufficient data to classify; investigation continues or case retained |

---

## Technology

| Capability | Purpose |
|---|---|
| **SIEM** | Log aggregation, correlation, detection rules, alerting, case management |
| **EDR** | Endpoint telemetry, threat detection, isolation, investigation |
| **SOAR** | Security Orchestration, Automation, and Response — automate repetitive triage and enrichment |
| **Threat Intelligence Platform** | Manage, consume, and produce threat intelligence |
| **Ticketing / Case Management** | Track alert handling, investigation notes, and closure |
| **Network Detection** | Network flow analysis; IDS/IPS; packet capture |
| **Cloud Security Monitoring** | Cloud-native logging; CSPM; cloud detection rules |

See: [[SIEM - Overview]], [[EDR - Overview]]

---

## Governance

| Element | Detail |
|---|---|
| **Policies** | SOC charter; incident response policy; acceptable use; data handling |
| **Playbooks** | Documented procedures for specific alert types and incident scenarios |
| **SLAs** | Defined timelines for triage, escalation, containment, and closure |
| **Escalation paths** | Who is notified at each severity level; management notification criteria |
| **Metrics and reporting** | Regular reporting to management on SOC performance and security posture |
| **Continuous improvement** | Lessons learned from incidents and false positives fed back into detections and playbooks |

---

## Metrics

| Metric | Definition |
|---|---|
| **MTTD (Mean Time to Detect)** | Average time from event occurrence to alert generation |
| **MTTR (Mean Time to Respond)** | Average time from alert to containment or closure |
| **Alert volume** | Total alerts generated per period |
| **True positive rate** | Proportion of alerts confirmed as genuine incidents |
| **False positive rate** | Proportion of alerts that are false positives; high rates indicate detection tuning need |
| **Alert closure rate** | Proportion of alerts closed within SLA |
| **Escalation rate** | Proportion of alerts escalated to Tier 2 or IR |

See [[SOC - Metrics]] for detail.

---

## SOC Maturity Levels

| Level | Characteristics |
|---|---|
| **1** | Reactive only; no structured detection; ad-hoc response |
| **2** | Alert-driven response; basic SIEM; generic rules; no threat hunting |
| **3** | Risk-based detection priorities; playbooks; MTTD/MTTR tracked; some threat hunting |
| **4** | Threat-informed detection; automated enrichment; proactive hunting; SOAR integration |
| **5** | Continuous improvement; adversary-aligned detection; metrics-driven; full lifecycle management |

---

## Related Notes

- [[SOC - Metrics]] — detailed metrics and measurement
- [[SIEM - Overview]] — SIEM architecture and capabilities
- [[EDR - Overview]] — endpoint detection and response
- [[Detection Engineering - Lifecycle]] — how detections are built and maintained
- [[Incident Response - Lifecycle]] — IR within the SOC lifecycle
- [[Threat Hunting - Overview]] — proactive investigation beyond alert response
- [[Control - Security Logging and SIEM]] — the SIEM as a security control
- [[Control - Endpoint Detection and Response]] — EDR as a security control
