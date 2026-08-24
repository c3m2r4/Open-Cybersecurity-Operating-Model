---
title: Control - Encryption
category: Control
tags:
  - Encryption
  - DataSecurity
  - PreventiveControl
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Encryption

## Control Objective

Protect data confidentiality and integrity where appropriate by using approved encryption for data at rest and in transit.

## Risk Addressed

Data exposure, interception, lost device exposure, unauthorized storage access.

## Control Type

Preventive. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Define encryption requirements by data classification and system type. Include key management, rotation, access control, backup encryption, and exception handling.

## Evidence

Encryption configuration, certificate records, key management records, disk encryption reports, storage encryption settings, exception approvals.

## Control Testing

Validate encryption is enabled for the defined population and that keys/certificates are managed according to standard.

## Failure Conditions

Unencrypted sensitive data, weak protocols, unmanaged keys, expired certificates, broad key access.

## Metrics

Encryption coverage, expired certificates, key access exceptions, unencrypted sensitive repositories.

## Related Notes

- [[Data Security Domain]]
- [[Control - Secrets Management]]
- [[Control Evidence Standard]]

