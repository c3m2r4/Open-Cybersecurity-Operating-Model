---
title: Control - Cloud Logging and Monitoring
category: Control
tags:
  - CloudSecurity
  - Logging
  - Monitoring
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Cloud Logging and Monitoring

## Control Objective

Collect and monitor cloud activity logs needed for detection, investigation, and response.

## Risk Addressed

Undetected cloud misuse, privilege abuse, data exposure, configuration changes, account compromise.

## Control Type

Detective. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Define required cloud log sources by provider and service, protect log integrity, centralize high-value logs, monitor administrative and data access events.

## Evidence

Cloud logging configuration, log source inventory, SIEM ingestion status, alert rules, retention settings, incident records.

## Control Testing

Validate required logs are enabled, retained, protected, and available for investigation.

## Failure Conditions

Missing audit logs, disabled logging, insufficient retention, unmonitored admin activity, mutable logs.

## Metrics

Cloud log source coverage, critical alert coverage, log ingestion failures, high-risk cloud detections.

## Related Notes

- [[Cloud Security Domain]]
- [[Security Monitoring Domain]]
- [[Control - Security Logging and SIEM]]

