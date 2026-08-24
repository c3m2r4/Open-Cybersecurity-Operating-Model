---
title: Offensive Security - Index
category: Index
tags:
  - OffensiveSecurity
  - SecurityValidation
  - Methodology
date_created: 2026-08-24
status: MAINTAINED
---

# Offensive Security — Index

Offensive security is the discipline of authorized testing that identifies weaknesses before adversaries do. Its purpose is not to demonstrate technical skill — it is to produce evidence that informs risk decisions and control improvements.

> A technically successful attack technique that has no path to business impact, is already detected, and is covered by an effective compensating control is a low priority. A simple misconfiguration that enables lateral movement to a critical system is a high priority.

## Operating Model Position

```text
Business Risk
  → Security Control
  → Attack Scenario (what could an adversary do?)
  → Authorized Security Validation
  → Evidence
  → Findings
  → Risk Rating
  → Remediation
  → Retest
  → Residual Risk
  → Management Decision
```

See [[Cybersecurity Operating Model]] for the full platform chain.

## Methodology

Start here before executing any testing:

| Note | Purpose |
|---|---|
| [[Offensive Security Methodology]] | 17-phase professional methodology from authorization to lessons learned |
| [[Security Testing Types]] | Taxonomy: scanning vs assessment vs pentest vs red team vs purple team vs control validation |
| [[Offensive Security Safety and Authorization]] | Lab vs authorized assessment vs production scope. Rules of engagement. |
| [[Risk-Based Offensive Security]] | Why business risk matters more than technical impressiveness |
| [[Offensive Security Reporting]] | Converting evidence into professional findings and executive communication |

## Testing Domains

| Domain | Purpose |
|---|---|
| [[Reconnaissance - Concepts]] | Understand exposed attack surface before scope is opened |
| [[Enumeration - Concepts]] | Identify services, identities, permissions, and misconfigurations |
| [[Vulnerability Assessment - Concepts]] | Confirm weaknesses and prioritize by risk |
| [[Web Application Security - Overview]] | Authentication, authorization, injection, access control, API, business logic |
| [[API Security - Overview]] | REST/GraphQL security: auth, exposure, injection, rate limiting |
| [[Active Directory Security - Overview]] | Identity, Kerberos, NTLM, ADCS, delegation, trusts, attack paths |
| [[Privilege Escalation - Concepts]] | Local and domain paths from low privilege to higher privilege |
| [[Lateral Movement - Concepts]] | Network movement, credential reuse, segmentation validation |
| [[Cloud Security Testing - Overview]] | IAM misconfigurations, storage exposure, compute risks, logging gaps |
| [[Red Team and Adversary Simulation - Overview]] | Objective-based adversary emulation |

## Learning Path

### Beginner

> Start with the why, not the how.

1. Read [[Cybersecurity Operating Model]] — understand where validation fits
2. Read [[Security Testing Types]] — learn the vocabulary correctly
3. Read [[Offensive Security Safety and Authorization]] — understand authorization before anything else
4. Read [[Risk-Based Offensive Security]] — understand what makes a finding matter

### Practitioner

> Understand the methodology before executing techniques.

1. Read [[Offensive Security Methodology]] end to end
2. Study the testing domain most relevant to your current work
3. Read [[Offensive Security Reporting]] to understand expected output
4. Connect findings to [[Control Failure Workflow]] and [[Control Testing Methodology]]

### Security Professional

> Testing is only valuable when connected to detection and remediation.

1. Review methodology and add detection validation to every engagement phase
2. Integrate findings with [[07 - PURPLE TEAM/Purple Team - Index]]
3. Ensure every finding connects to a control in [[Security Controls - Index]]
4. Use [[GOAD Project - Risk and Control Bridge]] for AD validation
5. Use [[Mythic Lab - Project Bridge]] for C2 and post-exploitation telemetry

### Risk Professional

> You do not need to know how to execute attacks. You need to understand what findings mean for risk.

1. Read [[Risk-Based Offensive Security]]
2. Read [[Offensive Security Reporting]] — focus on the finding template fields: Business Context, Impact, Likelihood, Risk, Residual Risk
3. Read the Management View section in [[Security Validation - Management View]]

## Project Links

| Project | Role in Offensive Security |
|---|---|
| [[GOAD Project - Risk and Control Bridge]] | Authorized Active Directory laboratory — identity, credential, and path testing |
| [[Mythic Lab - Project Bridge]] | Authorized C2 and post-exploitation laboratory — payload, callback, and detection telemetry |
| [[VAPT - Project Bridge]] | Assessment methodology framework — scoping, reporting, retest |
| [[Operation Cypher-Knife - Project Bridge]] | Authorized operation case study — evidence, findings, reporting, lessons learned |

## Templates

| Template | Use |
|---|---|
| [[Template - Professional Finding (Full)]] | Professional finding with all required fields |
| [[Template - Penetration Test Finding]] | Rapid finding capture |
| [[Template - Control Assessment]] | Link findings to control assessment |
| [[Template - Security Topic Lifecycle]] | Full lifecycle from threat to management decision |
