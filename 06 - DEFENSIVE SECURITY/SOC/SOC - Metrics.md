---
title: SOC - Metrics
category: Security Operations
tags:
  - SOC
  - Metrics
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# SOC — Metrics

## Purpose

SOC metrics measure whether security operations is functioning effectively. Good metrics are actionable — they identify where improvement is needed and track whether changes are having effect. Poor metrics measure activity (number of alerts) without measuring outcome (detection effectiveness, response quality).

> Do not establish target values without organizational context. What constitutes good MTTD or MTTR depends on the organization's risk tolerance, attack surface, and operational maturity. Do not import target values from benchmarks without adjusting them to your environment.

---

## Core Operational Metrics

### MTTD — Mean Time to Detect

| Field | Detail |
|---|---|
| **Definition** | The average time between the occurrence of a security event and the generation of an alert for that event |
| **Why It Matters** | Shorter MTTD reduces attacker dwell time and limits potential damage |
| **Measurement** | (Sum of detection times for all events in period) / (count of events) |
| **Influencing Factors** | Log collection latency; correlation rule coverage; detection logic quality; log retention |
| **Limitation** | MTTD is meaningless if detection coverage is low — a fast detection of one technique obscures the absence of detection for others |

---

### MTTR — Mean Time to Respond

| Field | Detail |
|---|---|
| **Definition** | The average time from alert generation to containment or closure of the incident |
| **Why It Matters** | Faster response limits the adversary's ability to achieve their objective |
| **Measurement** | (Sum of response times for all incidents in period) / (count of incidents) |
| **Influencing Factors** | Playbook quality; escalation efficiency; authorization to act; tool capability (EDR isolation); staffing |
| **Limitation** | Average can be distorted by outliers — track percentiles (P50, P90, P99) for more meaningful measurement |

---

### Alert Volume and Trend

| Metric | Meaning |
|---|---|
| Total alerts per period | Absolute volume; track trend to identify unusual spikes |
| Alerts by source | Which data sources or detection rules generate the most alerts |
| Alerts by severity | Distribution across Critical / High / Medium / Low |
| Alert volume trend | Week-over-week or month-over-month change |

---

### Detection Effectiveness

| Metric | Definition | Why It Matters |
|---|---|---|
| **True positive rate** | Confirmed incidents / total alerts | High rate = detections are reliable |
| **False positive rate** | False positive alerts / total alerts | High rate = detection quality is poor; analyst time is wasted |
| **Benign true positive rate** | Authorized activity triggering alerts / total alerts | May indicate need for allowlisting |
| **Alert-to-incident escalation rate** | Escalated alerts / total alerts | Indicates severity of activity in environment |
| **Detection coverage** | Percentage of known attack techniques for which a detection exists | Structural view of detection gaps |

---

### Response Quality

| Metric | Definition |
|---|---|
| **Escalation time** | Time from Tier 1 alert receipt to Tier 2 escalation |
| **Containment time** | Time from incident declaration to containment action |
| **SLA compliance rate** | Percentage of alerts or incidents handled within defined SLA |
| **Closure rate** | Percentage of alerts closed within defined timeframe |
| **Repeat incident rate** | Percentage of incidents involving a previously identified and remediated issue |

---

### Log Source Coverage

| Metric | Definition |
|---|---|
| **Log source inventory** | Count of distinct data sources feeding the SIEM |
| **Log source availability** | Percentage of expected log sources with active ingestion |
| **Critical asset coverage** | Percentage of critical assets with log collection active |
| **Log latency** | Time from event occurrence to availability in SIEM for investigation |

---

## What Metrics Should Drive

| Metric Observation | Improvement Action |
|---|---|
| High false positive rate from a specific rule | Detection tuning; review rule logic; add exclusions |
| High MTTD for a specific technique | Review detection coverage; add or improve rule |
| Low log source coverage for critical assets | Expand log collection; fix broken log sources |
| Long escalation time | Review triage process; improve playbooks; training |
| Repeat incidents from same root cause | Root cause remediation was incomplete; re-examine control effectiveness |

---

## Metrics That Are Not Useful Alone

| Metric | Why It Can Mislead |
|---|---|
| **Total alerts per period** | High volume may reflect poor detection quality (false positives), not high attack activity |
| **Number of incidents closed** | Closing incidents quickly may mean closing before investigation is complete |
| **Number of hunting queries run** | Activity metric; the finding and outcome of hunting matters more |
| **Percentage of critical alerts** | Percentage is meaningless without knowing the quality of the criticality classification |

---

## Reporting to Management

SOC metrics for management reporting should answer:

| Management Question | Metric |
|---|---|
| Are we getting faster at detecting threats? | MTTD trend |
| Are we getting faster at responding? | MTTR trend |
| Are our detections working? | True positive rate; detection coverage |
| Are our analysts overwhelmed? | Alert volume; false positive rate |
| Did we improve after the last incident? | Repeat incident rate; SLA compliance |
| Are we logging the right things? | Log source coverage; critical asset coverage |

---

## Related Notes

- [[SOC - Operating Model]] — SOC people, process, technology, and governance
- [[Detection Engineering - Lifecycle]] — how detection quality is maintained
- [[Security Validation Metrics]] — full metrics across offensive, defensive, and purple domains
