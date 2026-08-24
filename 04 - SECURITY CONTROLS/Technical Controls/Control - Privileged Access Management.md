---
title: Control - Privileged Access Management
category: Control
tags:
  - PAM
  - PrivilegedAccess
  - IAM
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Privileged Access Management

## Control Objective

Reduce the risk that high-impact privileges are misused, stolen, shared, or left unmanaged.

## Risk Addressed

Domain compromise, cloud tenant compromise, administrative misuse, credential theft, and unauthorized configuration changes.

## Control Type

Preventive and detective. Administrative and technical.

## Control Owner

VERIFY.

## Implementation Guidance

Inventory privileged accounts, require MFA, separate administrative and standard accounts, use vaulting where appropriate, monitor privileged sessions, define emergency access, and review privileges regularly.

## Evidence

Privileged account inventory, PAM configuration, vault records, MFA coverage, session logs, access approvals, emergency access records, review evidence.

## Control Testing

Confirm privileged population, enrollment in controls, review completion, and monitoring evidence. Validate break-glass accounts are controlled and tested.

## Failure Conditions

Shared admin accounts, unmanaged domain/cloud admins, no MFA, stale privileged access, no session logs, unreviewed emergency accounts.

## Metrics

Privileged coverage, unmanaged privileged accounts, emergency account use, privileged access review exceptions.

## Related Notes

- [[Identity and Access Management]]
- [[GOAD Project - Risk and Control Bridge]]
- [[Control - Multi-Factor Authentication]]

