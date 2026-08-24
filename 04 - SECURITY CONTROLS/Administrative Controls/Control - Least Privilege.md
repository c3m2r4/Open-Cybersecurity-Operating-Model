---
title: Control - Least Privilege
category: Control
tags:
  - LeastPrivilege
  - IAM
  - PreventiveControl
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Least Privilege

## Control Objective

Limit access to the minimum privileges required for authorized business activity.

## Risk Addressed

Privilege misuse, lateral movement, data exposure, excessive access, insider risk, and account compromise impact.

## Control Type

Preventive. Administrative and technical.

## Control Owner

VERIFY.

## Implementation Guidance

Use role-based access, approval workflows, access reviews, privileged access controls, segregation of duties, just-in-time access where appropriate, and service-account ownership.

## Evidence

Role definitions, access approvals, entitlement exports, group membership reports, privileged access inventory, access review records, exception approvals.

## Control Testing

Verify that access maps to business need and that excessive access is removed within the defined process. Sampling and frequency are organization-specific: VERIFY.

## Failure Conditions

Excessive privileges, orphaned access, privilege granted without approval, no owner, no review, service accounts with broad permissions.

## Metrics

Privileged account count, excessive access findings, access review completion, dormant accounts, unowned service accounts.

## Related Notes

- [[Identity and Access Management]]
- [[Control - Access Review]]
- [[Control - Privileged Access Management]]

