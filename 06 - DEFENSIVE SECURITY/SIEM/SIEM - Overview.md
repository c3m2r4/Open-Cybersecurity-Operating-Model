---
title: SIEM - Overview
category: Security Operations
tags:
  - SIEM
  - Detection
  - Logging
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# SIEM — Overview

## Purpose

A Security Information and Event Management (SIEM) system is the central nervous system of defensive operations. It aggregates logs from across the enterprise, normalizes them, correlates events to detect threats, and provides a platform for analysts to investigate alerts.

> A SIEM is only as valuable as the quality of the data it ingests and the logic of the detection rules running on it. A SIEM without tuned rules is just an expensive search engine.

---

## Core Architecture

| Component | Function |
|---|---|
| **Data Collection** | Agents, API connectors, or syslog listeners that pull or receive logs from source systems |
| **Parsing / Normalization** | Converting raw logs (e.g., Windows Event Log, AWS CloudTrail) into a common schema (e.g., source IP, destination IP, username) |
| **Storage / Retention** | Hot storage for rapid searching; cold storage for compliance and long-term retention |
| **Correlation Engine** | The rule engine that evaluates incoming events against detection logic in real time |
| **Alerting / Case Management** | Generation of alerts and routing them to analysts or SOAR platforms |
| **Dashboarding / Reporting** | Visualizations for SOC metrics and operational visibility |

---

## Log Collection Priorities

You cannot log everything. Prioritize data sources that provide high-value security telemetry:

| Priority | Data Source | What It Provides |
|---|---|---|
| **High** | Domain Controllers | Authentication, AD object changes, privilege use |
| **High** | Identity Providers (Okta, Entra ID) | Cloud authentication, MFA events, SSO |
| **High** | EDR | Process creation, network connections, file modifications |
| **High** | Cloud Control Plane (CloudTrail, Azure Activity) | Cloud resource creation, IAM changes, network changes |
| **High** | Network Perimeter (Firewalls, VPNs) | Remote access authentication, blocked traffic, egress connections |
| **Medium** | Web Proxies / DNS | Internal client outbound traffic, C2 beacons |
| **Medium** | Email Security Gateways | Phishing indicators, malware attachment logs |
| **Medium** | Key Application Logs | Authentication and authorization for critical business apps |
| **Low** | Full Packet Capture / NetFlow | Very high volume; often better handled by dedicated NDR |
| **Low** | Workstation System Logs | Often redundant if EDR is deployed; extremely high volume |

---

## Normalization and Schema

Correlation requires normalization. A SIEM must understand that `src_ip`, `SourceIp`, and `c-ip` all mean the same thing.

If data is not mapped to a common schema (like Elastic ECS, Splunk CIM, or OSSEM), you cannot write a single detection rule for "Brute Force" that covers Windows, Linux, and Cloud authentication simultaneously.

---

## Correlation and Detection

A correlation rule looks for specific conditions or sequences of events across normalized data.

### Detection Types

| Type | Example | Complexity |
|---|---|---|
| **Atomic Indicator** | Alert if `destination_ip == 192.168.1.50` | Low (Threat Intel matching) |
| **Threshold** | Alert if `failed_logon_count > 10` within `5 minutes` from the same `source_ip` | Medium (Brute Force) |
| **Sequence** | Alert if `VPN_logon` is followed by `RDP_connection` to a sensitive server within `2 minutes` | High (Lateral Movement) |
| **Behavioral / Anomaly** | Alert if `user` accesses a system they have not accessed in the last `30 days` | Very High (Requires baselining) |

See: [[Detection Engineering - Lifecycle]], [[SIEM - Detection Use Cases]]

---

## Alerting and Tuning

### False Positives

A false positive occurs when a detection rule triggers on benign activity. Excessive false positives cause alert fatigue, leading analysts to ignore real alerts.

**Tuning workflow:**
1. Identify false positive pattern (e.g., vulnerability scanner triggering "Brute Force" alert)
2. Verify activity is genuinely benign and authorized
3. Add exclusion to the rule logic (e.g., `AND source_ip != 'Scanner_IP'`)
4. Document the exclusion

### Alert Fatigue

If a SIEM generates more alerts than the SOC can triage, the organization has a tuning problem, not a staffing problem.
- Route low-fidelity alerts to dashboards for hunting, not to the analyst queue.
- Only generate interrupt-driven alerts for events that require immediate human investigation.

---

## Connectivity to Other Domains

The SIEM sits at the center of the defensive operating model:

| Domain | Integration |
|---|---|
| [[SOC - Operating Model]] | SIEM is the primary tool for Tier 1 triage and investigation |
| [[EDR - Overview]] | EDR sends high-fidelity alerts and telemetry to the SIEM |
| [[Incident Response - Lifecycle]] | SIEM provides the historical data required for scoping an incident |
| [[Threat Intelligence - Overview]] | IOCs from threat intelligence are fed into SIEM matching rules |
| [[05 - OFFENSIVE SECURITY/Offensive Security - Index]] | Offensive testing validates that the SIEM actually detects attacks |

---

## Related Notes

- [[SIEM - Detection Use Cases]] — how to build and structure correlation rules
- [[Detection Engineering - Quality Model]] — how to evaluate rule quality
- [[Control - Security Logging and SIEM]] — the SIEM as a formal security control
