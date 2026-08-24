---
title: Data Security Domain
category: Information Security Domain
tags:
  - DataSecurity
  - Encryption
  - DLP
date_created: 2026-08-24
status: MAINTAINED
---

# Data Security Domain

## Executive Summary

Data security protects the confidentiality, integrity, and availability of information throughout its lifecycle.

## Beginner

Data security means knowing what information you have, where it lives, who owns it, who can access it, how long it should be kept, and how it should be protected or destroyed.

## Core Principles

| Principle | Meaning |
|---|---|
| Confidentiality | Data is accessible only to authorized parties |
| Integrity | Data remains accurate, complete, and trustworthy |
| Availability | Data is accessible when needed for authorized business use |

## Practitioner

Core practices include data classification, data ownership, discovery, lifecycle management, encryption at rest, encryption in transit, key management, DLP, retention, secure disposal, backup protection, data leakage monitoring, and sensitive information handling.

## Security Professional

Threats include unauthorized access, accidental disclosure, insider misuse, ransomware, data corruption, insecure sharing, weak encryption, poor key management, and excessive retention.

## Controls

| Control Objective | Related Control |
|---|---|
| Protect data confidentiality | [[Control - Encryption]] |
| Detect or prevent leakage | [[Control - Data Loss Prevention]] |
| Limit access to sensitive data | [[Control - Least Privilege]] |
| Preserve recoverability | [[Control - Backup Protection]] |
| Govern retention and disposal | Administrative policy - VERIFY |

## Evidence

Evidence includes classification records, data owner approvals, encryption configuration, key management records, DLP alerts, backup reports, retention schedules, disposal records, access review evidence, and incident tickets.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| Data classification coverage | KCI | Scope - VERIFY |
| Encryption coverage | KCI | Must define systems and data classes |
| DLP incidents | KRI | Requires tuning and classification context |
| Backup success rate | KCI | Should be paired with restore testing |
| Sensitive data access exceptions | KRI | Requires owner approval |

## Management View

Data security affects regulatory exposure, customer trust, operational continuity, and competitive sensitivity. Management decisions often involve classification scope, encryption/key management investment, DLP tolerance, retention, and backup resilience.

## Related Notes

- [[Control - Encryption]]
- [[Control - Data Loss Prevention]]
- [[Risk Management Methodology]]

