---
title: Control - Secure SDLC
category: Control
tags:
  - SecureSDLC
  - ApplicationSecurity
  - DevSecOps
date_created: 2026-08-24
status: MAINTAINED
---

# Control - Secure SDLC

## Control Objective

Integrate security requirements, design review, testing, remediation, and release risk decisions into software delivery.

## Risk Addressed

Insecure design, vulnerable code, exposed secrets, vulnerable dependencies, weak authorization, insufficient logging.

## Control Type

Preventive. Administrative and technical.

## Control Owner

VERIFY.

## Implementation Guidance

Define security requirements, threat modeling triggers, SAST/DAST/SCA expectations, code review, secrets scanning, remediation workflow, and release exception process.

## Evidence

Threat models, security requirements, scan results, code review records, dependency reports, remediation tickets, exception approvals.

## Control Testing

Validate that required security activities occurred for in-scope releases and that high-risk findings were remediated or formally accepted.

## Failure Conditions

No threat model for high-risk changes, unresolved critical findings, secrets in code, no release risk decision.

## Metrics

Security test completion, critical findings beyond SLA, threat model coverage, secrets found, accepted release exceptions.

## Related Notes

- [[Application Security Domain]]
- [[Application and DevSecOps - Index]]
- [[VAPT - Project Bridge]]

