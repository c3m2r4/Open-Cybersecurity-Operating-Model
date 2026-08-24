---
title: Control - Secrets Management
category: Control
tags:
  - SecretsManagement
  - ApplicationSecurity
  - CloudSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Secrets Management

## Control Objective

Protect credentials, keys, tokens, certificates, and other secrets from unauthorized access or exposure.

## Risk Addressed

Credential leakage, unauthorized system access, cloud compromise, application compromise.

## Control Type

Preventive. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Use approved secret stores, avoid hard-coded secrets, restrict access, rotate secrets, monitor use, and define emergency rotation.

## Evidence

Secret store configuration, access policies, scan results, rotation records, incident tickets, exception approvals.

## Control Testing

Validate secret storage location, access control, rotation evidence, and absence of secrets in repositories or artifacts.

## Failure Conditions

Secrets in code, shared credentials, stale keys, broad access, no owner, no rotation process.

## Metrics

Secrets found in repositories, unrotated secrets, unowned secrets, high-risk secret access exceptions.

## Related Notes

- [[Application Security Domain]]
- [[Cloud Security Domain]]
- [[Control - Encryption]]

