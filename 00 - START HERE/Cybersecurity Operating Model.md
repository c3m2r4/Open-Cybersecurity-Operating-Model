---
title: Cybersecurity Operating Model
category: Operating Model
tags:
  - OperatingModel
  - Risk
  - Controls
  - Cybersecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Cybersecurity Operating Model

This model connects enterprise risk thinking with practical security validation. It is the main reasoning chain for the vault.

## Executive Summary

Cybersecurity work becomes management-relevant when it is connected to business objectives, assets, threats, controls, validation evidence, residual risk, and decisions. Technical testing is valuable because it proves whether a control works, not merely because a technique succeeds in a lab.

```text
Business Risk
  -> IT Risk Scenario
  -> Security Control
  -> Implementation
  -> Control Test
  -> Offensive Validation
  -> Detection Validation
  -> Incident Response
  -> Remediation
  -> Residual Risk
  -> Management Decision
```

## Example: MFA

| Layer | Example |
|---|---|
| Business Risk | Unauthorized account access |
| IT Risk Scenario | Credential compromise allows access to sensitive systems |
| Control | Multi-factor authentication |
| Policy | MFA required for privileged, remote, and high-risk access |
| Implementation | Identity provider, conditional access, enrollment, recovery |
| Control Test | Confirm MFA coverage and bypass exceptions |
| Offensive Validation | Test phishing-resistant gaps, prompt fatigue, legacy protocols |
| Detection Validation | Confirm alerts for impossible travel, MFA fatigue, suspicious enrollment |
| Incident Response | Disable account, revoke sessions, rotate credentials, review logs |
| Residual Risk | Remaining risk after control and monitoring are operating |
| Management Decision | Accept, mitigate, transfer, or avoid |

## Core Concepts

| Concept | Meaning |
|---|---|
| Business Objective | The outcome the organization needs to protect, such as revenue, safety, service availability, legal obligation, or trust |
| Asset | The system, data, identity, process, or service that supports the business objective |
| Threat | The actor, event, or condition that could cause harm |
| Vulnerability | The weakness that makes harm more likely or more severe |
| Risk Event | A plausible event where a threat exploits a vulnerability and affects an asset |
| Control | The safeguard that prevents, detects, corrects, deters, or compensates for the risk |
| Validation | Evidence-based checking that the control is designed and operating as expected |
| Residual Risk | The risk that remains after controls and treatment actions |
| Management Decision | The accountable choice to mitigate, accept, transfer, or avoid risk |

## Risk Treatment Options

| Option | Meaning | Example |
|---|---|---|
| Mitigate | Reduce likelihood or impact through controls | Enforce MFA for privileged access |
| Accept | Formally tolerate residual risk | Approve a time-bound exception with compensating controls |
| Transfer | Shift part of the impact to another party | Cyber insurance or contractual allocation |
| Avoid | Stop the risky activity | Retire an exposed service that is not business-critical |

## Evidence Standard

A claim should be supported by evidence appropriate to the claim:

| Claim | Evidence |
|---|---|
| A control exists | Policy, standard, configuration, architecture diagram, system record |
| A control works | Test result, log, screenshot, ticket, monitoring output, control assessment |
| A weakness exists | Finding, scan output, manual validation, configuration review |
| A risk was accepted | Approval record, risk owner, expiry, compensating controls |
| A lab technique was validated | Authorized lab note, scope, command output or observation, cleanup notes |

If evidence is missing, mark the item `VERIFY` or `NOT YET ESTABLISHED`.

## Project Mapping

| Project | Platform Role |
|---|---|
| [[GOAD Project - Risk and Control Bridge]] | Technical validation of Active Directory risks and controls |
| [[Mythic Lab - Project Bridge]] | C2, post-exploitation, and defensive telemetry learning |
| [[Operation Cypher-Knife - Project Bridge]] | Evidence, findings, reporting, and lessons learned |
