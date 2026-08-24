---
title: MITRE ATT&CK - Overview
category: Framework
tags:
  - MITRE
  - ATTACK
  - Framework
  - PurpleTeam
date_created: 2026-08-24
status: MAINTAINED
---

# MITRE ATT&CK — Overview

## Purpose

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is a globally accessible knowledge base of adversary tactics and techniques based on real-world observations. In the Cybersecurity Operating Model, ATT&CK serves as the common language (the Rosetta Stone) connecting Offensive Security, Defensive Security, and Threat Intelligence.

> Without a common language, the Red Team writes a report about "Dumping LSASS," the Blue Team writes a rule for "Credential Access," and Threat Intel warns about "Mimikatz." ATT&CK standardizes this to T1003.001 (OS Credential Dumping: LSASS Memory).

---

## Core Structure

ATT&CK is structured hierarchically:

### 1. Tactics (The "Why")
The tactical objective the adversary is trying to achieve. There are 14 tactics in the Enterprise matrix.
*Examples:* Initial Access, Execution, Persistence, Privilege Escalation, Lateral Movement, Exfiltration.

### 2. Techniques (The "How")
The specific method the adversary uses to achieve the tactical objective.
*Examples:* Phishing (to achieve Initial Access); Scheduled Task (to achieve Execution/Persistence).

### 3. Sub-Techniques (The "Specific How")
A more specific description of the technique.
*Example:* Phishing: Spearphishing Attachment (T1566.001).

### 4. Procedures (The "Exact Implementation")
The specific tool, malware, or command line used by an adversary to execute the technique.
*Example:* APT29 using PowerShell to execute a specific payload.

---

## How the Operating Model Uses ATT&CK

ATT&CK is the integration point across the platform:

| Domain | How it uses ATT&CK |
|---|---|
| **[[Threat Intelligence - Overview]]** | Profiles threat actors by the Techniques (T-Codes) they are known to use. |
| **[[Offensive Security Methodology]]** | Maps Red Team engagement findings to the Techniques executed. |
| **[[Detection Engineering - Lifecycle]]** | Maps every detection rule to the specific Technique it is designed to detect. |
| **[[Purple Team - Lifecycle]]** | Uses ATT&CK matrices as the scorecard to track validation coverage. |

---

## The Concept of Coverage

A common metric is "ATT&CK Coverage" — the percentage of techniques an organization can detect or prevent.

**Warning: Coverage is Nuanced.**
1. **You cannot achieve 100% coverage.** Many techniques rely on legitimate system administration behavior. Blocking them breaks the business.
2. **Coverage is not binary.** Having one detection rule mapped to T1003.001 (LSASS Memory) does not mean you are "covered" against all procedures an attacker might use to dump LSASS.

### Prioritizing Coverage

Do not try to detect the entire matrix. Prioritize based on:
1. **Threat Intel:** Which techniques are relevant threat actors actually using?
2. **Crown Jewels:** Which techniques are required for an attacker to reach your critical assets?
3. **Choke Points:** Which techniques are unavoidable for attackers? (e.g., almost all Windows attacks require some form of Execution and Credential Access).

---

## ATT&CK Navigator

The ATT&CK Navigator is the visual tool used by Purple Teams to heat-map an organization's posture.

- **Red Mapping:** Techniques successfully executed during tests.
- **Blue Mapping:** Techniques with active, validated detection rules.
- **Overlap:** Validated security posture.
- **Gaps:** Areas requiring detection engineering or control improvement.

See: [[Attack Detection Response Matrix]] for how this is implemented internally.

---

## Related Notes

- [[Attack Detection Response Matrix]] — the internal implementation of ATT&CK
- [[Detection Engineering - Quality Model]] — how detection quality maps to TTPs (Pyramid of Pain)
- [[Threat Intelligence - Overview]] — profiling actors using ATT&CK
