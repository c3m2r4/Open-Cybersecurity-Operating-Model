---
title: Control - Firewall and Traffic Filtering
category: Control
tags:
  - Firewall
  - NetworkSecurity
  - EgressFiltering
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Firewall and Traffic Filtering

## Control Objective

Permit authorized network traffic and deny unnecessary or risky traffic.

## Risk Addressed

Unauthorized access, exposed services, command-and-control egress, data exfiltration, lateral movement.

## Control Type

Preventive and detective. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Define ingress and egress requirements, use least-permissive rules, review high-risk rules, log relevant denies/allows, and maintain exception expiry.

## Evidence

Firewall rule exports, change records, rule review records, traffic logs, exception approvals, test results.

## Control Testing

Review rule design and validate selected allowed/blocked traffic paths. Sample size and cadence: VERIFY.

## Failure Conditions

Any-to-any rules, unmanaged internet exposure, no rule owner, stale exceptions, missing logging.

## Metrics

High-risk rules, expired exceptions, internet-exposed services, rule review completion.

## Related Notes

- [[Network Security Domain]]
- [[Control - Network Segmentation]]
- [[Security Monitoring Domain]]

