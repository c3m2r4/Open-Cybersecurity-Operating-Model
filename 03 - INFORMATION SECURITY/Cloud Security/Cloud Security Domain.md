---
title: Cloud Security Domain
category: Information Security Domain
tags:
  - CloudSecurity
  - IAM
  - Monitoring
date_created: 2026-08-24
status: MAINTAINED
---

# Cloud Security Domain

## Executive Summary

Cloud security protects cloud identities, networks, storage, workloads, configurations, logs, and data. Cloud security must account for shared responsibility between the cloud provider and the customer.

## Beginner

In cloud environments, the provider secures some parts of the platform, but the customer still has responsibilities such as access, configuration, data protection, logging, and workload security.

## Practitioner

Core practices include shared responsibility analysis, IAM, cloud networking, storage security, secrets, encryption, logging, monitoring, configuration management, CSPM concepts, workload security, container security, and cloud incident response.

## Security Professional

Threats include excessive cloud permissions, public storage exposure, insecure network paths, leaked access keys, weak logging, misconfigured workloads, vulnerable containers, insecure CI/CD integrations, and insufficient incident response readiness.

## Controls

| Control Objective | Related Control |
|---|---|
| Govern cloud permissions | [[Control - Least Privilege]] |
| Detect cloud activity | [[Control - Cloud Logging and Monitoring]] |
| Protect cloud data | [[Control - Encryption]] |
| Reduce misconfiguration | [[Control - Secure Configuration]] |
| Protect secrets | [[Control - Secrets Management]] |

## Evidence

Evidence includes cloud account inventory, IAM policy exports, logging configuration, storage access configuration, encryption settings, key management records, CSPM findings, workload scan results, container scan results, and incident response runbooks.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| Public storage exceptions | KRI | Requires data classification context |
| Cloud admin MFA coverage | KCI / KRI | Target - VERIFY |
| Critical CSPM findings | KRI | Provider/tool specific - VERIFY |
| Logging coverage | KCI | Must define required log sources |
| Unrotated secrets or keys | KRI | Requires inventory and ownership |

## Management View

Cloud security affects speed, resilience, data exposure, and operational accountability. Management decisions often involve landing zone standards, identity governance, logging costs, tooling, and exception acceptance.

## Related Notes

- [[Security Architecture - Index]]
- [[Application and DevSecOps - Index]]
- [[Control - Cloud Logging and Monitoring]]

