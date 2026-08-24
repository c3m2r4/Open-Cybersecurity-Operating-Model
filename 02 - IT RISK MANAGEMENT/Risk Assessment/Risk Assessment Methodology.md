---
title: Risk Assessment Methodology
category: Risk Management
tags:
  - RiskAssessment
  - Methodology
date_created: 2026-08-24
status: MAINTAINED
---

# Risk Assessment Methodology

## Purpose

This note explains how to assess cybersecurity risk in a defensible way without fabricating precision.

## Assessment Workflow

```text
Define Scope
  -> Identify Asset
  -> Define Risk Event
  -> Identify Threat and Vulnerability
  -> Estimate Inherent Risk
  -> Identify Existing Controls
  -> Assess Control Effectiveness
  -> Estimate Residual Risk
  -> Recommend Treatment
  -> Identify Evidence
  -> Obtain Decision
```

## Likelihood Considerations

Assess likelihood through observable drivers:

| Driver | Examples |
|---|---|
| Exposure | Internet-facing, internal-only, privileged access path, third-party access |
| Threat activity | Known targeting, commodity attack pattern, insider scenario |
| Vulnerability condition | Missing patch, weak configuration, excessive access, process gap |
| Exploitability | Ease of exploitation, prerequisites, required privileges |
| Control strength | Prevention, detection, response, monitoring, compensating controls |

## Impact Considerations

Assess impact through business consequences:

| Impact Type | Examples |
|---|---|
| Financial | Loss, fraud, recovery cost, contractual penalty |
| Operational | Service disruption, manual workaround, degraded capability |
| Legal / Regulatory | Investigation, notification, enforcement, contractual breach |
| Reputational | Loss of trust, customer concern, public scrutiny |
| Safety | Harm to people or safety-critical operations |
| Data | Confidentiality, integrity, availability, privacy impact |

## Control Effectiveness

| Rating Concept | Meaning |
|---|---|
| Design effective | The control, if operated as written, would address the risk |
| Operating effective | Evidence shows the control is operating as intended |
| Partially effective | The control reduces risk but has meaningful gaps |
| Ineffective | The control does not materially reduce the risk |
| Not tested | Effectiveness is unknown |

Do not rate control effectiveness without evidence.

## Output

A complete risk assessment should support a decision:

| Decision Element | Requirement |
|---|---|
| Risk owner | Named or marked `VERIFY` |
| Recommended treatment | Mitigate, accept, transfer, or avoid |
| Evidence | Linked or requested |
| Target date | Required for mitigation |
| Acceptance authority | Required for acceptance |
| Residual risk | Explained without unsupported precision |

## Related Notes

- [[Template - Risk Assessment]]
- [[Risk Treatment and Acceptance]]
- [[Control Testing in Risk Management]]

