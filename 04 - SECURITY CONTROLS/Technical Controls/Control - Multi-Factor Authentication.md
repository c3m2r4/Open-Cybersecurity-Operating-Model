---
title: Control - Multi-Factor Authentication
category: Control
tags:
  - MFA
  - IAM
  - PreventiveControl
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Multi-Factor Authentication

## Control Objective

Reduce the likelihood that stolen or guessed credentials alone can be used to access systems.

## Risk Addressed

Account compromise, credential theft, phishing, password reuse, privileged access misuse.

## Control Type

Preventive. Technical.

## Control Owner

VERIFY.

## Implementation Guidance

Prioritize privileged access, remote access, administrative consoles, cloud applications, email, and sensitive data access. Define exception handling, recovery procedures, and break-glass handling.

## Evidence

MFA policy exports, coverage reports, conditional access rules, exception register, privileged-account coverage, enrollment records, authentication logs.

## Control Testing

Design effectiveness asks whether MFA scope covers the intended risk. Operating effectiveness asks whether MFA was enforced for the defined population and period.

## Failure Conditions

Unprotected privileged accounts, legacy authentication bypass, unmanaged exceptions, weak recovery process, missing monitoring for enrollment or repeated prompts.

## Metrics

MFA coverage, privileged MFA coverage, exception count, failed/high-risk authentication trends. Targets: VERIFY.

## Detection and Response

Defenders should monitor suspicious login patterns, MFA fatigue indicators, new device enrollment, impossible travel, legacy protocol use, and session/token anomalies.

## Residual Risk

MFA does not eliminate phishing, token theft, social engineering, malware, or insider misuse. Stronger factors may be required for high-risk access.

## Related Notes

- [[Identity and Access Management]]
- [[2FA - Risk and Control Bridge]]
- [[Risk Treatment and Acceptance]]

