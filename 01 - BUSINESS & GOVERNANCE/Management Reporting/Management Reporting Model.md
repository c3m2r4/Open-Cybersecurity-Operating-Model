---
title: Management Reporting Model
category: Governance
tags:
  - ManagementReporting
  - Executive
  - Risk
date_created: 2026-08-24
status: MAINTAINED
---

# Management Reporting Model

## Executive Summary

Management reporting should translate security evidence into decisions. A good report explains exposure, control gaps, treatment options, progress, residual risk, and support required.

## Reporting Translation

```text
Technical Finding
  -> Business Risk
  -> Business Impact
  -> Control Gap
  -> Treatment Options
  -> Cost / Tradeoff
  -> Residual Risk
  -> Decision Required
```

## Core Reporting Questions

| Question | Answer Should Include |
|---|---|
| What happened or what changed? | Finding, incident, control failure, threat, metric movement |
| Why does it matter? | Business process, asset, impact category |
| What is the exposure? | Scope, likelihood drivers, impact drivers, uncertainty |
| What is being done? | Treatment plan, owner, target date, status |
| What remains? | Residual risk, exceptions, unresolved dependencies |
| What decision is required? | Funding, risk acceptance, policy approval, prioritization, escalation |

## Useful Report Types

| Report | Audience | Purpose |
|---|---|---|
| Executive Cyber Risk Summary | Executives / board | Material risks, trends, decisions |
| Risk Treatment Status | Risk owners | Progress on mitigation, acceptance, overdue actions |
| Control Effectiveness Summary | Security and audit stakeholders | Control design and operation |
| Vulnerability Exposure Summary | IT and executives | High-risk exposure, SLA performance, blockers |
| Incident Lessons Learned | Management and operators | Impact, root cause, improvement actions |

## Avoid Vanity Metrics

Metrics should drive decisions. Counts without context usually do not.

| Weak Metric | Better Question |
|---|---|
| Number of vulnerabilities found | How much high-risk exposure remains on critical assets? |
| Number of alerts generated | How many meaningful incidents were detected and triaged effectively? |
| Number of users trained | Did high-risk behavior decrease or reporting improve? |
| Number of policies published | Are policy requirements implemented, tested, and owned? |

## Related Notes

- [[Cybersecurity Metrics Model]]
- [[Risk Reporting Model]]
- [[Template - Executive Summary]]

