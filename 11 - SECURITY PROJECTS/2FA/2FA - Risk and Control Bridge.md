---
title: 2FA - Risk and Control Bridge
category: Project Bridge
project: 2FA
tags:
  - MFA
  - IAM
  - Controls
  - Risk
date_created: 2026-08-24
status: DRAFT
---

# 2FA - Risk and Control Bridge

Use this note to document two-factor authentication as a business risk control, not just a technical setting.

## Lifecycle

| Layer | Notes |
|---|---|
| Business Risk | Unauthorized account access |
| Asset | User accounts, privileged accounts, remote access, cloud applications |
| Threat | Credential theft, phishing, password reuse, session hijacking |
| Vulnerability | Missing MFA, weak MFA, exception sprawl, legacy authentication |
| Control | MFA, conditional access, phishing-resistant authentication |
| Control Test | Coverage review, exception review, enrollment review |
| Offensive Validation | Bypass testing in authorized lab or approved engagement |
| Detection Validation | Suspicious MFA prompts, new device enrollment, impossible travel |
| Remediation | Remove exceptions, disable legacy protocols, require stronger factors |
| Residual Risk | Remaining risk after enforcement and monitoring |
| Management Decision | Accept residual risk or fund additional improvements |

