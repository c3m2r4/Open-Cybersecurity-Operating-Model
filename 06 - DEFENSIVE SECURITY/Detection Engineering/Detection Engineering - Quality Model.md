---
title: Detection Engineering - Quality Model
category: Security Operations
tags:
  - DetectionEngineering
  - Quality
  - SIEM
date_created: 2026-08-24
status: MAINTAINED
---

# Detection Engineering — Quality Model

## Purpose

Not all detection rules are equal. A rule that alerts on a specific malware file hash is fragile. A rule that alerts on the fundamental behavior of credential dumping is robust. This note defines the criteria for evaluating detection quality.

---

## The Pyramid of Pain

David Bianco's Pyramid of Pain describes how difficult it is for an adversary to change their indicators when blocked or detected. Detection engineering should aim as high up the pyramid as feasible.

| Level | Indicator Type | Example | Adversary Difficulty to Change | Detection Robustness |
|---|---|---|---|---|
| **Top** | TTPs (Behaviors) | Using WMI for lateral movement | **Tough** (Must learn new techniques) | **High** (Catches variations) |
| **↑** | Tools | Mimikatz, Cobalt Strike | **Challenging** (Must write/buy new tools) | **Medium** |
| **↑** | Network / Host Artifacts | Specific registry keys, HTTP User-Agents | **Annoying** (Requires configuration change) | **Medium-Low** |
| **↑** | Domain Names | `evil-c2.com` | **Simple** (Register new domain) | **Low** |
| **↑** | IP Addresses | `192.0.2.5` | **Easy** (Change proxy/VPS) | **Low** |
| **Base** | Hash Values | SHA256 of malware file | **Trivial** (Change one byte) | **Lowest** (Brittle) |

> Detection rules based on hashes or IPs are fragile. They detect yesterday's attack, not tomorrow's. Aim to detect the underlying behavior (the TTP).

---

## Precision vs. Recall (Signal-to-Noise)

Detection engineering balances two competing metrics:

| Metric | Definition | Security Context |
|---|---|---|
| **Precision** | Out of all the alerts generated, how many were actual attacks? | Low precision means alert fatigue (too much noise). |
| **Recall** | Out of all the actual attacks that occurred, how many did the rule catch? | Low recall means missed attacks (false negatives). |

* **High Precision, Low Recall:** The rule only fires when it is 100% certain it's an attack, but it misses variations. (e.g., Alerting only on `mimikatz.exe` by name).
* **High Recall, Low Precision:** The rule catches every possible variation of the attack, but alerts on lots of legitimate admin activity too. (e.g., Alerting anytime `powershell.exe` makes a network connection).

**The Goal:** Tuned behavioral rules that achieve high recall (catching the behavior) while maintaining acceptable precision (filtering out known legitimate activity via exclusions).

---

## Criteria for a High-Quality Detection

A high-quality rule meets these criteria:

1. **Behavioral:** It detects the technique, not just the specific tool.
2. **Documented:** It has a formal Use Case (Objective, Threat, Playbook).
3. **Actionable:** When it fires, the analyst knows exactly what to investigate.
4. **Tuned:** False positives from known legitimate business processes are excluded.
5. **Tested:** It has been proven to work against simulated attacks.
6. **Robust:** It survives minor variations in attacker execution.

---

## Evaluating Detection Failures

When an attack is missed, determine the failure point:

| Failure Point | Meaning | Solution |
|---|---|---|
| **Telemetry Gap** | The activity was not logged. | Enable log collection for that event type. |
| **Ingestion Gap** | The log was generated but not sent to the SIEM. | Fix log forwarding / routing. |
| **Logic Gap** | The log was in the SIEM, but the rule logic missed the variation. | Update the rule logic. |
| **Tuning Error** | The rule caught it, but an overly broad exclusion filtered it out. | Refine the exclusion. |
| **Triage Failure** | The alert fired, but the analyst closed it incorrectly. | Improve playbook and training. |

See: [[Purple Team - Lessons Learned Model]] for how this feeds back into the loop.

---

## Related Notes

- [[Detection Engineering - Lifecycle]] — the process of building rules
- [[SIEM - Detection Use Cases]] — documenting the rule
- [[SOC - Metrics]] — tracking false positive rates
