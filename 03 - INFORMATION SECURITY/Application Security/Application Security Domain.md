---
title: Application Security Domain
category: Information Security Domain
tags:
  - ApplicationSecurity
  - SecureSDLC
  - DevSecOps
date_created: 2026-08-24
status: MAINTAINED
---

# Application Security Domain

## Executive Summary

Application security reduces risk by building and operating software with secure design, secure code, tested dependencies, protected secrets, controlled deployment, and useful logging.

## Beginner

Applications process business data and user actions. Application security helps make sure software does not expose data, trust the wrong user, accept malicious input, or leak secrets.

## Practitioner

Core practices include Secure SDLC, threat modeling, secure architecture, code review, SAST, DAST, SCA, secrets management, dependency management, API security, authentication, authorization, input validation, logging, and secure deployment.

## Security Professional

Threats include broken access control, injection, insecure design, secrets exposure, vulnerable dependencies, insecure APIs, weak session management, insufficient logging, and insecure CI/CD paths.

## Controls

| Control Objective | Related Control |
|---|---|
| Build security into delivery | [[Control - Secure SDLC]] |
| Protect credentials and tokens | [[Control - Secrets Management]] |
| Detect vulnerable dependencies | [[Control - Vulnerability Management Process]] |
| Validate application risk | [[Control - Security Logging and SIEM]] |
| Reduce unauthorized access | [[Control - Least Privilege]] |

## Validation

Validation may include architecture review, threat modeling, code review, SAST, DAST, SCA, API testing, configuration review, secrets scanning, and authorized offensive testing. Link detailed testing to VAPT or project-specific notes rather than duplicating commands.

## Evidence

Evidence includes threat models, architecture review records, pipeline scan results, code review records, dependency reports, secrets scan results, DAST results, remediation tickets, exception approvals, and deployment records.

## Metrics

| Metric | Type | Notes |
|---|---|---|
| Critical app findings beyond SLA | KRI | Requires severity methodology |
| SCA critical dependency exposure | KRI | Business context matters |
| Secrets found in repositories | KRI | Must include remediation and rotation |
| Threat model coverage | KCI | Scope - VERIFY |
| Security test completion for releases | KPI / KCI | Do not treat completion as proof of security |

## Management View

Application security reduces risk in customer-facing services, internal workflows, and data processing. Management decisions often involve delivery tradeoffs, remediation funding, release risk acceptance, and secure SDLC investment.

## Related Notes

- [[Application and DevSecOps - Index]]
- [[VAPT - Project Bridge]]
- [[Control - Secure SDLC]]

