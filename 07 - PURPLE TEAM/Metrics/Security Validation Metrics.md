---
title: Security Validation Metrics
category: Metrics
tags:
  - Validation
  - Metrics
  - PurpleTeam
date_created: 2026-08-24
status: MAINTAINED
---

# Security Validation Metrics

## Purpose

Validation metrics measure the ROI of the security budget. If an organization spends millions on EDR and SIEM, validation metrics prove whether those tools actually detect and prevent attacks. Good validation metrics drive engineering priorities; poor validation metrics create a false sense of security.

> Measuring the number of alerts the SOC closes is an operational metric (activity). Measuring the percentage of simulated attacks the SOC successfully detects is a validation metric (outcome).

---

## Core Validation Metrics

### 1. Detection Coverage (ATT&CK Coverage)

| Field | Detail |
|---|---|
| **Definition** | The percentage of prioritized MITRE ATT&CK techniques for which the organization has a validated detection rule. |
| **Why It Matters** | Shows structural readiness against known threat actor behavior. |
| **Measurement** | (Number of prioritized techniques with validated rules) / (Total number of prioritized techniques) |
| **Limitation** | Coverage is not absolute. Detecting one variation of a technique does not mean all variations are covered. |

### 2. Validation Pass Rate (Control Effectiveness)

| Field | Detail |
|---|---|
| **Definition** | The percentage of simulated attacks that are successfully prevented or detected according to the expected baseline. |
| **Why It Matters** | Proves that deployed controls are functioning as designed and have not suffered configuration drift. |
| **Measurement** | (Number of tests passed) / (Total number of tests executed in a period) |
| **Limitation** | Only as good as the tests. If the tests are too simple, the pass rate will be artificially high. |

### 3. Time to Detection Tuning (TTDT)

| Field | Detail |
|---|---|
| **Definition** | The average time it takes to create, test, and deploy a new detection rule after a gap is identified (e.g., via a Purple Team exercise or Threat Intel report). |
| **Why It Matters** | Measures the agility of the detection engineering capability. |
| **Measurement** | Time from gap identification to validated rule deployment in production. |
| **Limitation** | Rushing deployment to meet metrics can result in high false-positive rates. |

### 4. Telemetry Gap Ratio

| Field | Detail |
|---|---|
| **Definition** | The percentage of failed detection tests where the root cause was missing telemetry (logs not collected). |
| **Why It Matters** | Directs budget and engineering effort toward foundational logging infrastructure rather than complex rule writing. |
| **Measurement** | (Failed tests due to missing logs) / (Total failed tests) |

---

## Dwell Time (The Ultimate Metric)

**Dwell Time** is the duration an adversary (or Red Team) operates in an environment before being detected and contained.

* **MTTD (Mean Time to Detect) + MTTR (Mean Time to Respond) = Dwell Time**

During Red Team engagements and Adversary Simulations, Dwell Time is the most critical metric to report to management.

| Scenario | Dwell Time Result | Implication |
|---|---|---|
| Red Team executes initial access and is blocked by EDR immediately. | Minutes | Controls are highly effective at the perimeter. |
| Red Team gains access, moves laterally for 14 days, and achieves objective undetected. | 14+ Days | Significant detection failure. In a real incident, the data would be gone. |

---

## What Metrics Should Drive

| Metric Observation | Action |
|---|---|
| Low Validation Pass Rate (Controls failing) | Halt new tool deployment; focus engineering on fixing existing controls. |
| High Telemetry Gap Ratio | Pause detection rule creation; focus on deploying logging agents and fixing SIEM ingestion. |
| High Detection Coverage but Long Dwell Time | The rules exist, but the SOC triage process is failing (ignoring alerts). Improve playbooks and SOC training. |

---

## Related Notes

- [[Control Validation - Framework]] — how the tests generating these metrics are run
- [[SOC - Metrics]] — operational metrics (MTTD, MTTR)
- [[Security Validation - Management View]] — how to report these metrics to leadership
