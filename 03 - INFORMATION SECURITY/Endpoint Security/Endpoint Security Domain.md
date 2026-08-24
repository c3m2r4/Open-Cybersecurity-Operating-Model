---
title: Endpoint Security Domain
category: Information Security Domain
tags:
  - EndpointSecurity
  - EDR
  - PatchManagement
date_created: 2026-08-24
status: MAINTAINED
---

# Endpoint Security Domain

## Executive Summary

Endpoint security protects workstations, servers, and other endpoint devices from compromise, misuse, malware, insecure configuration, and operational disruption.

## Beginner

An endpoint is a device people or services use, such as a laptop, workstation, or server. Endpoint security keeps those devices hardened, monitored, patched, and recoverable.

## Practitioner

Core practices include endpoint inventory, secure configuration, EDR, anti-malware, host firewall, application control, patch management, local administrator management, disk encryption, USB/removable media controls, endpoint telemetry, isolation, and containment.

## Security Professional

Endpoint failures can lead to malware execution, credential theft, lateral movement, persistence, data theft, and ransomware. Endpoint security connects directly to vulnerability management, EDR, SOC operations, and incident response.

```text
Endpoint Security
  -> Vulnerability Management
  -> EDR
  -> SOC
  -> Incident Response
```

## Controls

| Control Objective | Related Control |
|---|---|
| Detect endpoint compromise | [[Control - Endpoint Detection and Response]] |
| Reduce exploitable weaknesses | [[Control - Patch Remediation]] |
| Enforce secure baseline | [[Control - Secure Configuration]] |
| Prevent unauthorized local privilege | [[Control - Least Privilege]] |
| Restore endpoint operations | [[Control - Incident Response Process]] |

## Evidence

Evidence includes asset inventory, EDR coverage reports, anti-malware status, patch compliance reports, secure baseline results, disk encryption reports, local admin group reports, isolation records, and incident tickets.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| EDR coverage | KCI | Coverage must account for critical servers and user endpoints |
| Patch compliance | KPI / KCI | Target - VERIFY |
| Unsupported endpoint count | KRI | Strong risk signal |
| Local admin exception count | KRI | Requires business justification |
| Endpoint isolation time | KPI | Useful during incident response |

## Management View

Endpoint compromise can disrupt operations, expose data, and become an entry point for wider compromise. Management may need to decide on tooling coverage, remediation prioritization, device lifecycle funding, or risk acceptance for unsupported systems.

## Related Notes

- [[Vulnerability Management Domain]]
- [[Security Monitoring Domain]]
- [[Incident and Resilience - Index]]

