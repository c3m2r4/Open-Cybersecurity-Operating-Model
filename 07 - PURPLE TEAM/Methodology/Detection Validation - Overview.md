---
title: Detection Validation - Overview
category: Methodology
tags:
  - Validation
  - DetectionEngineering
  - PurpleTeam
date_created: 2026-08-24
status: MAINTAINED
---

# Detection Validation — Overview

## Purpose

Detection Validation is the specific subset of Control Validation focused on ensuring that SIEM rules, EDR behavioral alerts, and SOC playbooks function exactly as intended when exposed to real attacker activity.

> A detection rule that is written, deployed, and never triggered by a simulated attack is a theoretical rule. Theoretical rules fail during real incidents.

---

## Why Detections Fail (The Case for Validation)

Detection logic breaks constantly, often silently. Validation is required because:

1. **Log Schema Changes:** The vendor updates the firewall OS, changing the format of the syslog output. The SIEM parser breaks; logs are no longer normalized; the detection rule stops firing.
2. **Log Source Failure:** A network change blocks syslog traffic from a domain controller. The logs stop arriving. The SIEM rule is syntactically perfect, but there is no data to alert on.
3. **Over-Tuning:** A SOC analyst adds an exclusion to a rule to filter out a noisy backup service, but uses a wildcard that is too broad, inadvertently filtering out malicious activity.
4. **Attacker Evolution:** The detection rule looks for `powershell.exe -enc`, but the attacker switches to `pwsh.exe -e`. The behavior is the same, but the brittle logic fails.

---

## Validation Methods

### 1. Atomic Testing (Unit Testing)

Executing a single, specific command to trigger a single detection rule.

| Detail | Description |
|---|---|
| **Tool Example** | Red Canary Atomic Red Team |
| **Use Case** | Validating that a newly written SIEM rule works before moving it to production. |
| **Pros** | Fast, repeatable, scriptable, highly specific. |
| **Cons** | Lacks context; does not test the SOC's ability to correlate events. |

### 2. Scenario Testing (Integration Testing)

Executing a sequence of actions that represent a partial attack path.

| Detail | Description |
|---|---|
| **Tool Example** | Custom scripts, Breach and Attack Simulation (BAS) tools |
| **Use Case** | Validating that a sequence of alerts (e.g., initial access + discovery + privilege escalation) triggers correlation rules and is properly triaged. |
| **Pros** | Tests correlation and SOC triage context. |
| **Cons** | Requires more coordination; harder to automate fully. |

### 3. Continuous Automated Validation

Using platforms that constantly execute safe attacks in production.

| Detail | Description |
|---|---|
| **Tool Example** | AttackIQ, Picus, SafeBreach |
| **Use Case** | Preventing configuration drift by continuously validating critical detections (daily/weekly). |
| **Pros** | Ensures long-term reliability; detects silent log failures immediately. |
| **Cons** | Expensive; requires careful deployment to avoid disrupting production. |

---

## The Validation Loop

1. **Detection Engineer** creates a new rule (e.g., "Detect unusual registry modification").
2. **Detection Engineer** creates a corresponding Atomic Test for that specific registry modification.
3. **Purple Team** executes the Atomic Test.
4. **Validation Result:**
   - *Pass:* The alert fired in the SIEM with the correct severity. Rule is approved for production.
   - *Fail:* No alert fired. Telemetry gap or logic error identified and fixed.
5. **Automation:** The Atomic Test is added to the continuous automated validation schedule.

---

## Detection Validation vs. SOC Triage

Validation is not just about the SIEM firing an alert. It must validate the entire pipeline:

```text
Action Executed (Atomic Test)
  ↓
Log Generated (Endpoint)
  ↓
Log Forwarded (Agent/Syslog)
  ↓
Log Ingested & Parsed (SIEM)
  ↓
Rule Matches (SIEM)
  ↓
Alert Generated (SIEM)
  ↓
Alert Routed to Queue (Ticketing/SOAR)
  ↓
Analyst Triages Successfully (Human)
```

If the alert fires but the playbook is so confusing that the analyst closes it as a false positive, the detection has failed.

---

## Related Notes

- [[Detection Engineering - Lifecycle]] — building the rules that require validation
- [[Control Validation - Framework]] — the broader framework for all controls
- [[Purple Team - Lifecycle]] — executing validation manually
- [[Attack Detection Response Matrix]] — mapping tests to detections
