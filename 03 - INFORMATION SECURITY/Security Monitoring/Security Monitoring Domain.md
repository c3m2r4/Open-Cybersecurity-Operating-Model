---
title: Security Monitoring Domain
category: Information Security Domain
tags:
  - SecurityMonitoring
  - SIEM
  - Detection
date_created: 2026-08-24
status: MAINTAINED
---

# Security Monitoring Domain

## Executive Summary

Security monitoring collects and analyzes telemetry so suspicious activity can be detected, investigated, and responded to.

## Monitoring Chain

```text
Attack
  -> Telemetry
  -> Detection
  -> Investigation
  -> Response
```

## Beginner

Security monitoring is how defenders notice that something suspicious may be happening.

## Practitioner

Core practices include logging, SIEM, EDR, network telemetry, authentication logs, identity logs, application logs, cloud logs, alerting, detection engineering, threat hunting, case management, and response handoff.

## Security Professional

Monitoring fails when important log sources are missing, telemetry is not normalized, alerts lack context, detections are noisy, analysts lack playbooks, or response actions are not connected to incidents.

## Controls

| Control Objective | Related Control |
|---|---|
| Collect useful telemetry | [[Control - Security Logging and SIEM]] |
| Detect endpoint compromise | [[Control - Endpoint Detection and Response]] |
| Detect cloud activity | [[Control - Cloud Logging and Monitoring]] |
| Escalate suspicious activity | [[Control - Incident Reporting]] |
| Validate detection | [[Control Testing Methodology]] |

## Evidence

Evidence includes log source inventory, SIEM ingestion status, detection rules, alert records, analyst notes, case records, EDR telemetry, cloud audit logs, false positive tuning records, and detection test results.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| Log source coverage | KCI | Required sources - VERIFY |
| Detection coverage | KCI / KRI | Threat model dependent |
| Mean time to detect | KPI / KRI | Requires consistent incident timing |
| Mean time to respond | KPI | Depends on incident classification |
| Alert false positive rate | KPI / KCI | Must not drive suppression of important signals |

## Management View

Security monitoring reduces time to detect and respond. Management decisions often involve logging cost, SIEM/EDR coverage, SOC staffing, response authority, and acceptable monitoring gaps.

## Related Notes

- [[Defensive Security - Index]]
- [[Purple Team - Index]]
- [[Control - Security Logging and SIEM]]

