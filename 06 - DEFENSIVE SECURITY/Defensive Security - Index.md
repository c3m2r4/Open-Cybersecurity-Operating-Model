---
title: Defensive Security - Index
category: Index
tags:
  - DefensiveSecurity
  - SOC
  - Detection
  - IncidentResponse
date_created: 2026-08-24
status: MAINTAINED
---

# Defensive Security — Index

Defensive security is the operating model that detects attacks, investigates threats, responds to incidents, and provides continuous evidence that controls are working. Its output is not just incident response — it is the evidence loop that closes the security validation lifecycle.

## Position in the Operating Model

```text
Attack Scenario
  → Authorized Test or Real Attack
  → Telemetry
  → Detection
  → Alert
  → Triage
  → Investigation
  → Classification
  → Containment
  → Eradication
  → Recovery
  → Lessons Learned
  → Control Improvement
```

See [[Cybersecurity Operating Model]] for the full platform chain.

## Validation Questions

| Question | Evidence |
|---|---|
| Was the activity logged? | Raw event, endpoint telemetry, network flow, cloud audit log |
| Was the activity detected? | Alert, rule match, correlation, case creation |
| Was the alert triaged correctly? | Analyst notes, classification, severity |
| Was investigation sufficient? | Timeline, IOC analysis, scope confirmation |
| Was response effective? | Containment, eradication, recovery actions |
| What improved? | Detection tuning, playbook update, control change |

## Subdomain Index

| Domain | Purpose |
|---|---|
| [[SOC - Operating Model]] | People, process, technology, governance, and metrics for security operations |
| [[SOC - Metrics]] | MTTD, MTTR, detection coverage, alert quality |
| [[SIEM - Overview]] | Log collection, correlation, detection rules, use cases |
| [[SIEM - Detection Use Cases]] | Example use case structure and approach |
| [[EDR - Overview]] | Endpoint telemetry, detection, containment, and response |
| [[Detection Engineering - Lifecycle]] | Building, testing, deploying, and maintaining detections |
| [[Detection Engineering - Quality Model]] | What makes a detection good |
| [[Threat Hunting - Overview]] | Hypothesis-driven investigation without a preceding alert |
| [[Incident Response - Lifecycle]] | Preparation through post-incident review |
| [[Incident Response - Roles and Escalation]] | Who does what; escalation thresholds; communications |
| [[Digital Forensics - Overview]] | Evidence preservation, acquisition, analysis, and reporting |
| [[Threat Intelligence - Overview]] | Strategic, operational, tactical, and technical intelligence |

## Connection to Offensive Security

Every offensive security finding should be asked:

> "Would this have been detected?"

If not: detection engineering and threat hunting should address the gap. If yes: validate that the detection is reliable, not just theoretical.

See [[07 - PURPLE TEAM/Purple Team - Index]] for the integration of offensive and defensive validation.

## Connection to Controls

Defensive security is a set of controls in its own right. Each capability in this domain maps to the control library:

| Capability | Control |
|---|---|
| SIEM / Security Logging | [[Control - Security Logging and SIEM]] |
| EDR | [[Control - Endpoint Detection and Response]] |
| Incident Response process | [[Control - Incident Response Process]] |
| Incident Reporting | [[Control - Incident Reporting]] |
| Security Monitoring | [[Control - Cloud Logging and Monitoring]] |
