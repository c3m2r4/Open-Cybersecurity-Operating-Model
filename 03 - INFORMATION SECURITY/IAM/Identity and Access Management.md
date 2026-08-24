---
title: Identity and Access Management
category: Information Security Domain
tags:
  - IAM
  - Identity
  - AccessControl
  - Risk
date_created: 2026-08-24
status: MAINTAINED
---

# Identity and Access Management

## Executive Summary

Identity and Access Management (IAM) controls who can access systems, data, and services. IAM reduces risk from unauthorized access, credential theft, privilege misuse, dormant accounts, and weak access governance.

## Beginner

IAM answers three simple questions: who are you, should you be allowed in, and what are you allowed to do after you get in?

## Practitioner

Operational IAM includes identity lifecycle management, joiner/mover/leaver processes, authentication, MFA, password controls, role-based access control, least privilege, privileged access management, service account governance, non-human identity governance, access reviews, dormant-account handling, emergency accounts, segregation of duties, conditional access, and identity governance.

## Security Professional

Threat scenarios include phishing, credential stuffing, password reuse, MFA fatigue, token theft, privilege escalation, service-account abuse, excessive permissions, and unreviewed access. IAM failures frequently become attack paths into Active Directory, cloud platforms, SaaS applications, and administrative consoles.

## Controls

| Control Objective | Related Control |
|---|---|
| Verify user identity before access | [[Control - Multi-Factor Authentication]] |
| Limit access to business need | [[Control - Least Privilege]] |
| Govern privileged accounts | [[Control - Privileged Access Management]] |
| Remove stale access | [[Control - Access Review]] |
| Manage identity lifecycle | Joiner/mover/leaver process - VERIFY owner |

## Evidence

Useful evidence includes identity provider configuration, MFA coverage reports, access review records, privileged account inventories, dormant account reports, conditional access policies, service account ownership records, and ticket/change records.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| MFA coverage | KCI / KRI | Target depends on risk appetite - VERIFY |
| Privileged account MFA coverage | KCI / KRI | High-value control coverage |
| Access review completion | KPI / KCI | Must include quality, not just completion |
| Dormant account count | KRI | Indicates stale access exposure |
| Privileged account count | KRI | Must be interpreted with business context |

## Management View

IAM failures can enable unauthorized access to critical systems and data. Management decisions commonly involve funding MFA/PAM, enforcing access reviews, accepting exceptions, or prioritizing identity governance remediation.

## Related Notes

- [[Risk Management Methodology]]
- [[Control Testing Methodology]]
- [[GOAD Project - Risk and Control Bridge]]

