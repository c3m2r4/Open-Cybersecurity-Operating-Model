---
title: Control Testing in Risk Management
category: Risk Management
tags:
  - ControlTesting
  - RiskManagement
  - Evidence
date_created: 2026-08-24
status: MAINTAINED
---

# Control Testing in Risk Management

## Executive Summary

Control testing determines whether a safeguard is designed and operating effectively enough to reduce risk. It supports residual risk decisions.

## Control Test Structure

| Element | Description |
|---|---|
| Objective | What the test is meant to prove |
| Scope | Systems, processes, users, time period, or population tested |
| Authorization | Approval to perform the test, especially for intrusive validation |
| Prerequisites | Access, data, tools, approvals, environment |
| Test Procedure | Steps performed |
| Expected Result | What should happen if the control works |
| Observed Result | What actually happened |
| Evidence | Logs, screenshots, exports, tickets, observations |
| Failure Condition | What outcome means the control failed |
| Risk Impact | How the result changes risk understanding |
| Retest | How remediation will be validated |

## Control Categories

| Category | Description | Example |
|---|---|---|
| Preventive | Stops or reduces likelihood | MFA, network filtering |
| Detective | Identifies unwanted activity | SIEM alert, EDR detection |
| Corrective | Restores or remediates | Account disablement, patching |
| Deterrent | Discourages behavior | Warning banner, policy enforcement |
| Compensating | Reduces risk when primary control is absent | Manual review during temporary exception |

Controls may also be administrative, technical, physical, or a combination.

## Testing and Offensive Validation

Offensive validation should be authorized, scoped, logged, and tied to a control objective. In this vault, detailed lab procedures should remain in authoritative technical notes such as [[GOAD - Command Execution Matrix]] rather than being duplicated in risk notes.

## Related Notes

- [[Template - Control Assessment]]
- [[Source of Truth and Evidence Standard]]
- [[GOAD Project - Risk and Control Bridge]]

