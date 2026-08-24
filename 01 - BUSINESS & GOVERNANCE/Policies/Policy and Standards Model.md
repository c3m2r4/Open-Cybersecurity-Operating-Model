---
title: Policy and Standards Model
category: Governance
tags:
  - Policy
  - Standards
  - Controls
date_created: 2026-08-24
status: MAINTAINED
---

# Policy and Standards Model

## Overview

Policies define management intent. Standards define required implementation expectations. Procedures describe how work is performed. Evidence shows whether the requirement is operating.

## Document Hierarchy

| Level | Purpose | Example |
|---|---|---|
| Policy | States what must be true and why | Privileged access must be controlled and reviewed |
| Standard | Defines specific mandatory requirements | MFA required for privileged accounts |
| Procedure | Explains how to perform the work | Steps to enroll and review privileged MFA |
| Guideline | Provides recommended practice | Preferred secure configuration pattern |
| Evidence | Demonstrates operation | Access review record, configuration export, ticket |

## Control Linkage

Every significant policy requirement should map to:

```text
Requirement
  -> Control Objective
  -> Control Owner
  -> Implementation Standard
  -> Test Method
  -> Evidence
  -> Exception Process
```

## Common Mistakes

| Mistake | Consequence |
|---|---|
| Writing policy that cannot be tested | Control effectiveness cannot be demonstrated |
| Mixing aspirational goals with requirements | Teams cannot tell what is mandatory |
| No exception process | Risk is accepted informally and invisibly |
| No owner | Requirements age without maintenance |

## Current Vault Status

Organization-specific policies are `NOT YET ESTABLISHED`. Notes in this vault may describe policy models and example requirements, but they should not claim that a policy exists unless an approved policy note or source is linked.

## Related Notes

- [[Control Testing in Risk Management]]
- [[Template - Security Exception]]
- [[Cybersecurity Governance Model]]

