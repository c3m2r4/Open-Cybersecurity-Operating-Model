---
title: Control Evidence Standard
category: Control Methodology
tags:
  - Evidence
  - Controls
  - Assurance
date_created: 2026-08-24
status: MAINTAINED
---

# Control Evidence Standard

## Overview

Evidence supports claims about control design, operation, failure, remediation, and residual risk. Screenshots may be useful, but they are not automatically sufficient.

## Evidence Types

| Evidence Type | Example |
|---|---|
| Configuration evidence | Policy export, system setting, firewall rule, identity provider policy |
| System-generated evidence | Logs, audit trails, scan results, EDR telemetry |
| Access review evidence | Review record, reviewer decision, removal ticket |
| Tickets | Remediation, approval, exception, change request |
| Reports | Coverage report, compliance report, vulnerability report |
| Screenshots | Point-in-time human-readable evidence |
| Change records | Approved change, implementation record, rollback plan |
| Approval records | Risk acceptance, exception approval, policy approval |
| Test results | Control test result, retest result, validation observation |

## Evidence Quality

| Quality | Meaning |
|---|---|
| Relevant | Directly supports the claim being made |
| Reliable | Comes from a trustworthy source |
| Complete | Covers the required population and period |
| Accurate | Correctly represents the system or process |
| Traceable | Can be tied to system, owner, date, and test |
| Current | Reflects the period being assessed |
| Reproducible | Another qualified reviewer could validate it |

## Common Evidence Problems

| Problem | Risk |
|---|---|
| Screenshot without timestamp or scope | Cannot prove period or population |
| Report without source definition | Reliability is unclear |
| Ticket closed without retest | Remediation may be assumed but unverified |
| Manual attestation only | May not prove operating effectiveness |

## Related Notes

- [[Source of Truth and Evidence Standard]]
- [[Control Testing Methodology]]
- [[Risk Reporting Model]]

