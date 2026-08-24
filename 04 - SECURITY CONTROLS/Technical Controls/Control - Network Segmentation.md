---
title: Control - Network Segmentation
category: Control
tags:
  - NetworkSegmentation
  - NetworkSecurity
  - PreventiveControl
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Network Segmentation

## Control Objective

Limit unauthorized connectivity and reduce blast radius if a system is compromised.

## Risk Addressed

Lateral movement, unauthorized access, malware spread, data exposure, uncontrolled administrative paths.

## Control Type

Preventive. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Segment by business function, trust level, data sensitivity, administration plane, and exposure. Document allowed flows, exceptions, and review process.

## Evidence

Network diagrams, firewall rules, routing tables, VLAN/segment design, approved flows, segmentation test results, exception records.

## Control Testing

Validate that unauthorized paths are blocked and required business paths remain available. Testing production segmentation requires authorization.

## Failure Conditions

Flat network, broad any-to-any rules, undocumented exceptions, unmanaged admin paths, no egress controls.

## Metrics

Segmentation exceptions, unauthorized open paths, critical systems without segmentation, rule review completion.

## Related Notes

- [[Network Security Domain]]
- [[Security Architecture - Index]]
- [[GOAD Project - Risk and Control Bridge]]

