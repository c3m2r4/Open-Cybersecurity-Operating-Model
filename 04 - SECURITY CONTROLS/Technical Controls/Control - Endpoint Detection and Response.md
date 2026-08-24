---
title: Control - Endpoint Detection and Response
category: Control
tags:
  - EDR
  - EndpointSecurity
  - DetectiveControl
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Endpoint Detection and Response

## Control Objective

Detect, investigate, and support containment of suspicious endpoint activity.

## Risk Addressed

Malware, credential theft, persistence, lateral movement, ransomware, unauthorized execution.

## Control Type

Detective and corrective. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Deploy to defined endpoint populations, monitor health, define alert triage, enable containment capability where approved, and integrate with incident response.

## Evidence

EDR coverage report, sensor health, alert records, containment records, policy configuration, incident tickets, test results.

## Control Testing

Validate coverage, sensor health, alert generation, triage workflow, and response handoff. Use authorized test methods only.

## Failure Conditions

Missing agents, unhealthy sensors, disabled protection, untriaged alerts, no containment authority.

## Metrics

EDR coverage, unhealthy agents, high-severity alert triage time, containment time, false positive trends.

## Related Notes

- [[Endpoint Security Domain]]
- [[Security Monitoring Domain]]
- [[Incident and Resilience - Index]]

