---
title: Source of Truth and Evidence Standard
category: Standard
tags:
  - Evidence
  - SourceOfTruth
  - Validation
date_created: 2026-08-24
status: MAINTAINED
---

# Source of Truth and Evidence Standard

## Purpose

This note defines how the vault distinguishes facts, lab evidence, assumptions, and recommendations.

## Source Hierarchy

| Source Type | Trust Use |
|---|---|
| Existing validated vault notes | Use as local source of truth for the lab or project they cover |
| Official standards and vendor documentation | Use for framework, product, and configuration references |
| Lab evidence | Use only for the authorized lab scope where it was collected |
| Professional judgment | Use for recommendations, architecture decisions, and risk interpretation |
| Example content | Use for teaching; mark clearly as `EXAMPLE` |
| Assumptions | Use temporarily; mark `VERIFY` before relying on them |

## Evidence Types

| Evidence | Suitable For |
|---|---|
| Policy or standard | Control requirement exists |
| Configuration export | Control implementation state |
| Screenshot | Human-readable observation |
| Log or telemetry | Detection, response, or operational activity |
| Ticket or change record | Remediation workflow and ownership |
| Test output | Validation result |
| Risk approval | Acceptance authority and expiry |
| Architecture diagram | Design intent and dependencies |

## Lab Evidence Rule

GOAD, Mythic, and Operation Cypher-Knife evidence is useful for learning and controlled validation. It does not prove that a real organization has the same exposure or control failure.

Use this distinction:

| Statement | Correct Treatment |
|---|---|
| "This worked in GOAD" | Lab-verified in authorized lab |
| "This exists in production" | Requires production evidence |
| "This control is effective" | Requires defined test scope and result |
| "This organization is compliant" | Requires formal assessment evidence |

## Unknown Information

Use consistent markers:

| Marker | Meaning |
|---|---|
| `VERIFY` | Plausible but not confirmed |
| `NOT YET ESTABLISHED` | Required information does not exist in the vault yet |
| `EXAMPLE` | Teaching example, not a claim about a real environment |

