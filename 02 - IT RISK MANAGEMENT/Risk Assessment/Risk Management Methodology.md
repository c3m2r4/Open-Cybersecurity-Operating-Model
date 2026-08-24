---
title: Risk Management Methodology
category: Risk Management
tags:
  - RiskManagement
  - Methodology
  - Governance
date_created: 2026-08-24
status: MAINTAINED
---

# Risk Management Methodology

## Executive Summary

Risk management connects uncertainty to decisions. In cybersecurity, the goal is not to eliminate all risk. The goal is to understand material risk, operate effective controls, prioritize treatment, and make accountable decisions about residual risk.

## Core Risk Formula

The simple teaching model is:

```text
Risk = Likelihood x Impact
```

This is useful for explaining risk, but enterprise risk scoring may include additional factors such as exploitability, exposure, asset criticality, threat capability, control effectiveness, regulatory impact, and velocity. Do not invent numeric ratings without an approved methodology.

## Risk Statement Pattern

Use this structure:

```text
There is a risk that [threat] could exploit [vulnerability] affecting [asset],
resulting in [business impact].
```

Example:

```text
EXAMPLE: There is a risk that a phishing attacker could use stolen credentials
to access a cloud application without MFA, resulting in unauthorized data access
and operational disruption.
```

## Required Risk Elements

| Element | Description |
|---|---|
| Risk Event | The scenario being assessed |
| Threat | Who or what could cause harm |
| Vulnerability | The weakness or condition that enables the event |
| Asset | The system, identity, data, service, or process affected |
| Business Impact | The consequence to business objectives |
| Likelihood | How plausible the event is, based on exposure, threat, and control state |
| Inherent Risk | Risk before current controls are considered |
| Existing Controls | Current safeguards that reduce likelihood or impact |
| Control Effectiveness | Whether controls are designed and operating as intended |
| Residual Risk | Risk remaining after current controls |
| Risk Owner | Accountable person for risk treatment |
| Treatment | Mitigate, accept, transfer, or avoid |
| Target Date | Expected date for treatment completion |
| Acceptance Authority | Person or forum authorized to accept residual risk |
| Evidence | Records supporting the assessment and decision |

## Maturity Model

| Level | Description | Typical Evidence |
|---|---|---|
| Level 1 - Initial | Ad hoc, reactive, limited visibility | Informal issue lists |
| Level 2 - Developing | Basic policies and controls, inconsistent execution | Partial risk register, some owners |
| Level 3 - Defined | Standard process, documented ownership, consistent assessments | Defined methodology, regular reviews |
| Level 4 - Managed | Metrics, control testing, monitoring, residual risk decisions | KRI/KCI reporting, treatment tracking |
| Level 5 - Optimized | Continuous improvement, automation, threat-informed investment | Automated evidence, scenario analysis, strategic risk reduction |

Level 5 is not automatically the right target for every organization or control. Maturity should be risk-driven.

## Related Notes

- [[Risk Assessment Methodology]]
- [[Risk Register Operating Guide]]
- [[Cybersecurity Governance Model]]
- [[Source of Truth and Evidence Standard]]

