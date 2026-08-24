---
title: Cybersecurity Metrics Model
category: Risk Management
tags:
  - Metrics
  - KPI
  - KRI
  - KCI
date_created: 2026-08-24
status: MAINTAINED
---

# Cybersecurity Metrics Model

## Overview

Cybersecurity metrics should support decisions. Separate performance, risk, and control indicators.

## Metric Types

| Type | Meaning | Example |
|---|---|---|
| KPI | Key Performance Indicator; measures process performance | Percentage of critical patches completed within SLA |
| KRI | Key Risk Indicator; signals risk exposure or risk movement | Number of critical vulnerabilities overdue on critical assets |
| KCI | Key Control Indicator; signals control operation | Percentage of privileged accounts protected by MFA |

## Useful Metrics

| Metric | Type | Why It Matters |
|---|---|---|
| Mean Time to Detect | KPI / KRI | Shows detection speed and potential dwell time exposure |
| Mean Time to Respond | KPI | Shows response execution capability |
| Critical Vulnerability SLA | KPI / KRI | Shows whether high-risk weaknesses are remediated on time |
| Patch Compliance | KCI | Shows whether patching control is operating |
| MFA Coverage | KCI / KRI | Shows authentication control coverage and residual access risk |
| EDR Coverage | KCI | Shows endpoint visibility and response coverage |
| Privileged Account Coverage | KCI / KRI | Shows exposure around high-impact accounts |
| Open High-Risk Findings | KRI | Shows unresolved material exposure |
| Control Failure Rate | KCI / KRI | Shows reliability of key safeguards |
| Third-Party Risk Exposure | KRI | Shows risk from suppliers and external dependencies |

## Metric Quality Test

| Question | Good Metric Answer |
|---|---|
| What decision does this support? | Prioritization, funding, escalation, acceptance, remediation |
| Who owns the metric? | Named owner or `VERIFY` |
| What is the data source? | Defined source, not anecdote |
| What is the threshold? | Linked to risk appetite or operational objective |
| What happens when it breaches? | Escalation or action path |

## Related Notes

- [[Management Reporting Model]]
- [[Risk Reporting Model]]
- [[Risk Register Operating Guide]]

