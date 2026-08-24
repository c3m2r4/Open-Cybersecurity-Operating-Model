---
title: SIEM - Detection Use Cases
category: Security Operations
tags:
  - SIEM
  - Detection
  - UseCases
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# SIEM — Detection Use Cases

## Purpose

A detection use case is a formal definition of what a SIEM rule is trying to detect, how it works, what log sources it requires, and how an analyst should respond when it triggers.

> Never deploy a rule to a production SIEM without documenting its use case. Undocumented rules lead to inconsistent triage, ignored alerts, and unmaintainable SIEM environments.

---

## Anatomy of a Use Case

Every SIEM detection should have documentation covering the following fields:

### 1. Metadata

| Field | Detail |
|---|---|
| **Use Case ID** | Unique identifier (e.g., UC-IAM-001) |
| **Title** | Clear, descriptive name |
| **Author / Owner** | Who built and maintains it |
| **Status** | Development / Testing / Active / Deprecated |
| **MITRE ATT&CK** | Tactic and Technique mapping |

### 2. Detection Logic

| Field | Detail |
|---|---|
| **Objective** | What is this trying to detect? |
| **Threat Scenario** | Context of how an adversary would use this |
| **Required Log Sources** | e.g., Windows Security (Domain Controllers), CloudTrail |
| **Logic Summary** | Plain English description of the detection logic |
| **Query / Code** | The actual SPL, KQL, or query language string |

### 3. Operational Details

| Field | Detail |
|---|---|
| **Severity** | Critical / High / Medium / Low |
| **False Positive Potential** | What legitimate activity might trigger this? |
| **Tuning / Exclusions** | Documented exceptions (e.g., authorized vulnerability scanners) |
| **Response Playbook** | Step-by-step instructions for the analyst investigating the alert |
| **Validation Test** | How to safely trigger this rule to prove it works |

---

## Example: Impossible Travel (UC-IAM-002)

| Field | Value |
|---|---|
| **Title** | Impossible Travel — Geographically Distant Logons |
| **MITRE ATT&CK** | T1078 (Valid Accounts) |
| **Objective** | Detect compromised credentials being used by an adversary in a different location than the legitimate user |
| **Log Sources** | VPN logs, Identity Provider (Okta/Entra) logs |
| **Logic** | Alert when a single user account authenticates successfully from two different countries within a time window shorter than the flight time between them |
| **Severity** | High |
| **False Positives** | VPN usage routing traffic through different exit nodes; automated services using user credentials |
| **Response Playbook** | 1. Identify user and locations. 2. Contact user to verify travel/VPN usage. 3. If unverified, force password reset and revoke active sessions. 4. Review actions taken by the second session. |
| **Validation** | Authenticate via VPN node in Country A, immediately disconnect and authenticate via VPN node in Country B |

---

## Example: DCSync Attack (UC-AD-005)

| Field | Value |
|---|---|
| **Title** | DCSync — Directory Replication by Non-DC Account |
| **MITRE ATT&CK** | T1003.006 (OS Credential Dumping: DCSync) |
| **Objective** | Detect an attacker simulating a domain controller to extract password hashes from Active Directory |
| **Log Sources** | Windows Security Event Log (Domain Controllers) |
| **Logic** | Alert when Event ID 4662 contains access mask `0x100` and properties `{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}` or `{1131f6ad-9c07-11d1-f79f-00c04fc2dcd2}` AND the SubjectUserName does not end in `$` (is not a computer account) |
| **Severity** | Critical |
| **False Positives** | Azure AD Connect service accounts; legitimate third-party AD sync tools |
| **Response Playbook** | 1. Identify source account and IP. 2. Verify if the account is an authorized sync service. 3. If unauthorized, immediately disable account and contain source IP. 4. Declare major incident — domain compromise likely. |
| **Validation** | Run Mimikatz `lsadump::dcsync` from authorized lab machine against test account |

---

## Connecting to Validation

A use case is only theoretical until it is tested. Every use case should be validated by the Purple Team or through automated adversary simulation tools.

See:
- [[Detection Validation - Overview]] — how to test detections
- [[Purple Team - Lifecycle]] — collaborative validation
- [[Active Directory - Detection and Hardening]] — AD specific detection logic
