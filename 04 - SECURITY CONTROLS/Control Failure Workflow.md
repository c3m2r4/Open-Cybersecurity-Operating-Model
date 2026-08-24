---
title: Control Failure Workflow
category: Control Methodology
tags:
  - ControlFailure
  - Remediation
  - Risk
date_created: 2026-08-24
status: MAINTAINED
---

# Control Failure Workflow

## Purpose

This workflow explains what should happen when a control fails, partially fails, or cannot be evidenced.

## Workflow

```text
Control Failure
  -> Validate Evidence
  -> Confirm Root Cause
  -> Assess Risk
  -> Determine Scope
  -> Identify Compensating Controls
  -> Remediation
  -> Retest
  -> Residual Risk
  -> Closure / Risk Acceptance
```

## Failure Types

| Failure Type | Example |
|---|---|
| Design failure | Control does not address the risk even if operated correctly |
| Operating failure | Control is appropriate but did not run or was bypassed |
| Evidence failure | Control may exist, but evidence is incomplete or unreliable |
| Scope failure | Control covers only part of the intended population |
| Ownership failure | No accountable owner maintains the control |

## Required Analysis

| Question | Why It Matters |
|---|---|
| What failed? | Defines remediation |
| What evidence supports failure? | Prevents unsupported conclusions |
| What is the affected population? | Defines risk scope |
| What is the root cause? | Prevents repeat failure |
| Are compensating controls operating? | Supports residual risk assessment |
| Who accepts residual risk? | Ensures accountable decision-making |

## Related Notes

- [[Risk Treatment and Acceptance]]
- [[Control Testing Methodology]]
- [[Template - Security Exception]]

