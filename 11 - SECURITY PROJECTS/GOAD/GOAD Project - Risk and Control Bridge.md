---
title: GOAD Project - Risk and Control Bridge
category: Project Bridge
project: GOAD
tags:
  - GOAD
  - ActiveDirectory
  - Risk
  - ControlValidation
date_created: 2026-08-24
status: MAINTAINED
---

# GOAD Project - Risk and Control Bridge

GOAD is the technical laboratory for validating Active Directory risks, identity controls, detection coverage, and remediation decisions.

## Enterprise Chain

| Layer | GOAD Mapping |
|---|---|
| Business Risk | Unauthorized access to critical systems through identity compromise |
| IT Risk Scenario | Weak Active Directory configuration enables credential theft, privilege escalation, or lateral movement |
| Security Control | Privileged access management, Kerberos hardening, ADCS hardening, delegation governance, monitoring |
| Technical Validation | [[GOAD - Command Execution Matrix]] |
| Offensive Test | Enumeration, credential abuse, ACL abuse, ADCS, delegation, trust paths |
| Detection | SIEM, EDR, Windows Event Logs, authentication telemetry |
| Control Gap | Missing hardening, excessive privilege, weak monitoring, poor segmentation |
| Remediation | Configuration change, privilege reduction, detection rule, incident playbook update |
| Risk Rating | Based on likelihood, impact, exploitability, and control effectiveness |
| Executive Finding | Convert validated path into business impact and decision required |

## Existing GOAD Notes

| Note | Purpose |
|---|---|
| [[GOAD - Vault Index]] | Current GOAD navigation |
| [[GOAD - Command Execution Matrix]] | Authoritative lab procedure map |
| [[GOAD - Active Directory - Inside the Domain Workflow]] | Post-compromise navigation |
| [[GOAD - Final Validation Checklist]] | Pre-operation validation |

## Next Notes To Create

| Note | Purpose |
|---|---|
| GOAD - AD Risk Scenarios | Risk register entries for AD attack paths |
| GOAD - Detection Coverage Matrix | Expected telemetry for each technique |
| GOAD - Executive Findings | Management-ready findings from validated paths |

