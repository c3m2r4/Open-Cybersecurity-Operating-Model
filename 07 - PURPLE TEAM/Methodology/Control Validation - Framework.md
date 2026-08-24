---
title: Control Validation - Framework
category: Methodology
tags:
  - Validation
  - Controls
  - SecurityOperations
date_created: 2026-08-24
status: MAINTAINED
---

# Control Validation — Framework

## Purpose

Security controls (firewalls, EDR, MFA, SIEM rules) degrade over time due to configuration drift, software updates, and evolving attacker techniques. Control validation is the continuous process of empirically proving that a control functions as designed and mitigates the intended risk.

> Do not trust vendor data sheets or implementation sign-offs. If a control has not been tested against a simulated attack recently, assume it is failing.

---

## The Validation Framework

Every security control in the environment (see [[04 - SECURITY CONTROLS/Security Controls - Index]]) must be subject to validation. The framework dictates *how* it is validated.

### 1. Define the Expected State

Before testing a control, you must define exactly what it is supposed to do.

| Component | Example (EDR Control) | Example (MFA Control) |
|---|---|---|
| **Control Objective** | Prevent execution of known malware; detect malicious behavior. | Ensure high-assurance identity verification for remote access. |
| **Expected Telemetry** | Log process creation (EID 1) and network connections (EID 3) to SIEM. | Log successful and failed MFA prompts to SIEM. |
| **Expected Prevention** | Block execution of `mimikatz.exe`. | Block VPN connection if MFA is not approved. |
| **Expected Detection** | Alert on `powershell.exe` downloading a payload from an external IP. | Alert on multiple MFA push rejections (MFA fatigue). |

### 2. Design the Validation Test

Create a specific, repeatable test that safely simulates the threat the control is designed to mitigate.

| Testing Method | Best For | Example |
|---|---|---|
| **Unit Test** | Isolated, specific functionality. | Downloading the EICAR test file to trigger AV. |
| **Automated Simulation** | Continuous validation of known TTPs. | Using Breach and Attack Simulation (BAS) tools to run daily credential dumping tests. |
| **Purple Team Exercise** | Complex behavior; tuning new rules. | Collaborative execution of a lateral movement path. |
| **Red Team Engagement** | End-to-end response validation. | Undisclosed campaign to test the SOC's ability to correlate disparate alerts. |

### 3. Execute and Measure

Execute the test and measure the result against the Expected State.

**Measurement Outcomes:**
1. **Prevented:** The control stopped the action automatically. (Ideal for known-bad).
2. **Detected:** The action succeeded, but the control generated a high-fidelity alert. (Ideal for suspicious behavior).
3. **Logged:** The action succeeded and was recorded in telemetry, but no alert fired. (Requires detection engineering).
4. **Missed:** The action succeeded and no telemetry was generated. (Critical control failure).

### 4. Tuning and Remediation

If the measurement does not match the Expected State, remediation is required.

| Failure | Remediation Action |
|---|---|
| **Missed (No Telemetry)** | Fix configuration (e.g., enable Sysmon; fix log forwarding to SIEM). |
| **Logged but not Detected** | Detection Engineering (e.g., write a SIEM rule for the logged behavior). |
| **Detected but not Prevented** | Adjust enforcement policy (e.g., move EDR from "Audit" to "Block" mode for that behavior). |

---

## Validation Frequency

Validation must be continuous, not annual.

| Control Type | Validation Frequency | Validation Method |
|---|---|---|
| High-Fidelity Detections | Daily / Weekly | Automated BAS |
| EDR Prevention | Monthly | Automated BAS / Manual spot-check |
| Incident Response Playbooks | Quarterly | Tabletop Exercise / Purple Team |
| Overall Security Posture | Annually | Red Team Engagement |

---

## Connecting to Risk Management

Control validation is the empirical feedback loop for IT Risk Management.

| Risk Register State | Validation Outcome | Action |
|---|---|---|
| Control is listed as "Effective" | Validation proves it fails consistently. | Update Risk Register: Control Effectiveness downgraded. Residual Risk increased. |
| Control is listed as "Ineffective" | Validation proves tuning was successful and it now blocks the threat. | Update Risk Register: Control Effectiveness upgraded. Residual Risk decreased. |

---

## Related Notes

- [[Control Validation - Maturity Model]] — measuring the maturity of this framework
- [[Security Validation Metrics]] — how to measure outcomes
- [[Security Controls - Index]] — the controls being validated
- [[Purple Team - Lifecycle]] — the collaborative execution of validation
