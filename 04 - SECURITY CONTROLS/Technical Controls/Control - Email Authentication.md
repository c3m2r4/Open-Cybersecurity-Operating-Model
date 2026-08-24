---
title: Control - Email Authentication
category: Control
tags:
  - EmailSecurity
  - SPF
  - DKIM
  - DMARC
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Email Authentication

## Control Objective

Reduce spoofing and improve trust in email sender identity.

## Risk Addressed

Phishing, business email compromise, executive impersonation, domain spoofing.

## Control Type

Preventive and detective. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Implement and maintain SPF, DKIM, and DMARC for owned domains. DMARC policy state and enforcement depend on domain readiness: VERIFY.

## Evidence

DNS records, mail authentication reports, gateway configuration, DMARC aggregate reports, exception records.

## Control Testing

Validate DNS records, alignment, policy behavior, and monitoring of authentication failures.

## Failure Conditions

Missing records, permissive DMARC without plan, unmanaged sending services, no report review.

## Metrics

Domain coverage, DMARC policy state, authentication failure trends, unauthorized sender sources.

## Related Notes

- [[Email Security Domain]]
- [[Control - Security Awareness Training]]
- [[Control - Multi-Factor Authentication]]

