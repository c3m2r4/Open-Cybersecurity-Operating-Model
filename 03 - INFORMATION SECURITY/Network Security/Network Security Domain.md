---
title: Network Security Domain
category: Information Security Domain
tags:
  - NetworkSecurity
  - Segmentation
  - Monitoring
date_created: 2026-08-24
status: MAINTAINED
---

# Network Security Domain

## Executive Summary

Network security reduces risk by controlling connectivity, limiting attack paths, protecting administrative access, and generating network telemetry for detection and response.

## Beginner

Network security decides which systems can talk to each other, which traffic is allowed, and which activity should be watched.

## Practitioner

Core practices include network architecture, VLAN design, firewalls, DMZs, internal segmentation, ingress filtering, egress filtering, secure administration, remote access, VPN, wireless security, DNS security, network monitoring, and Zero Trust concepts.

## Security Professional

Threats include exposed management ports, flat networks, excessive east-west connectivity, weak remote access, insecure wireless access, DNS abuse, command-and-control egress, and lateral movement.

## Controls

| Control Objective | Related Control |
|---|---|
| Limit unauthorized inbound access | [[Control - Firewall and Traffic Filtering]] |
| Limit lateral movement | [[Control - Network Segmentation]] |
| Restrict outbound abuse paths | [[Control - Firewall and Traffic Filtering]] |
| Secure remote administration | [[Control - Secure Remote Access]] |
| Detect suspicious network behavior | [[Control - Security Logging and SIEM]] |

## Validation

Validation may include firewall rule review, segmentation testing, route review, remote access configuration review, wireless security assessment, DNS logging review, and authorized lab testing. Do not test production network boundaries without explicit authorization.

## Evidence

Evidence includes network diagrams, firewall rules, change records, VPN configuration, wireless configuration, DNS logs, flow logs, segmentation test results, and exception approvals.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| Internet-exposed critical services | KRI | Requires asset inventory and exposure validation |
| Firewall rule review completion | KPI / KCI | Quality depends on evidence and remediation |
| Remote access MFA coverage | KCI / KRI | Target - VERIFY |
| Segmentation exceptions | KRI | Should have owner and expiry |

## Management View

Weak network security can increase breach likelihood and blast radius. Management decisions often involve segmentation investment, remote access policy, exception acceptance, and network modernization.

## Related Notes

- [[Security Architecture - Index]]
- [[Control - Network Segmentation]]
- [[Control - Firewall and Traffic Filtering]]

