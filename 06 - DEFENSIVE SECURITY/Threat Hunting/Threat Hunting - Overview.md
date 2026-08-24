---
title: Threat Hunting - Overview
category: Security Operations
tags:
  - ThreatHunting
  - SOC
  - Investigation
date_created: 2026-08-24
status: MAINTAINED
---

# Threat Hunting — Overview

## Purpose

Threat hunting is the proactive, hypothesis-driven search for malicious activity that has evaded existing detection rules. It is not alert triage. If an analyst is investigating a SIEM alert, they are performing incident response/triage. If an analyst is searching log data based on an idea of what an attacker *might* be doing, they are threat hunting.

> Threat hunting assumes the environment is already compromised and the automated defenses missed it.

---

## Hypothesis-Driven Hunting

A threat hunt begins with a specific hypothesis. A hypothesis is not "Let's look for bad stuff."

| Bad Hypothesis | Good Hypothesis |
|---|---|
| "Let's look for malware." | "Adversaries may use WMI to execute payloads remotely. We should look for `WmiPrvSE.exe` spawning unusual child processes (e.g., `powershell.exe`, `cmd.exe`) across the server fleet in the last 7 days." |
| "Are there any hackers on the network?" | "Threat actors frequently use Rclone for data exfiltration. We should search for execution of unsigned binaries named `rclone.exe` or network connections to known cloud storage providers from systems that do not normally access them." |

A hypothesis is based on:
- Threat intelligence (How are threat actors operating right now?)
- Red team/Pentest findings (What worked during our last engagement?)
- Crown Jewels analysis (How would someone steal our most valuable data?)

---

## The Threat Hunting Lifecycle

```text
1. Form Hypothesis (Based on Intel / Past Incidents / Crown Jewels)
     ↓
2. Identify Data Sources (Do we have the logs needed to prove/disprove this?)
     ↓
3. Execute Query / Investigation (Search the SIEM / EDR data)
     ↓
4. Validate Findings (Filter out false positives and legitimate admin activity)
     ↓
5. Triage / Escalate (If malicious activity is found, declare an incident)
     ↓
6. Automate (Create a detection rule if the query logic is robust)
     ↓
7. Document Lessons Learned (Identify logging gaps or required control changes)
```

---

## Hunting vs. Detection Engineering

Threat hunting and detection engineering are deeply linked, but distinct activities.

| Threat Hunting | Detection Engineering |
|---|---|
| Proactive, human-driven search | Reactive, automated alerting |
| High noise tolerance (analyst can filter manually) | Low noise tolerance (must not overwhelm SOC) |
| Output: Findings, Incidents, or new Detection Ideas | Output: Automated SIEM/EDR rules |

> A successful threat hunt often ends by turning the hunt query into an automated detection rule. However, not all hunts can be automated. Some queries require human context to distinguish between a sysadmin and an attacker.

---

## What Makes a Good Hunt?

A threat hunt is successful if it achieves one of three outcomes:
1. **Finds Evil:** Identifies an active compromise.
2. **Improves Detection:** Results in a new, high-fidelity detection rule.
3. **Identifies Gaps:** Discovers that the organization lacks the logging or visibility required to prove or disprove the hypothesis. (This is a valuable finding!).

---

## Related Notes

- [[SOC - Operating Model]] — where threat hunting fits in the SOC
- [[Detection Engineering - Lifecycle]] — automating hunt findings
- [[Threat Intelligence - Overview]] — forming hypotheses based on intelligence
- [[SIEM - Overview]] — the primary tool for executing hunts
