---
title: Template - Purple Team Exercise
category: Template
tags:
  - Template
  - PurpleTeam
  - Validation
date_created: 2026-08-24
status: MAINTAINED
---

# Template — Purple Team Exercise

> **Instructions for use:** Use this template to document the planning, execution, and outcomes of a Purple Team exercise. Do not execute the exercise until the 'Planning' section is fully approved. Delete these instructions before finalizing.

---

## Exercise Summary

**Exercise ID:** [e.g., PTE-2026-Q3-01]
**Execution Date:** [YYYY-MM-DD]
**Target Environment:** [e.g., Corporate Production, Staging AD, AWS Dev]
**Status:** [Planned / In Progress / Completed / Closed]

**Participants:**
- **Exercise Lead:** [Name]
- **Red Team (Execution):** [Name(s)]
- **Blue Team (Detection/Triage):** [Name(s)]
- **Engineering (Tuning):** [Name(s)]

**Objective:**
- [e.g., "Validate and tune detection logic for T1003.001 (OS Credential Dumping) and T1558.003 (Kerberoasting) on Windows Server 2022 domain controllers."]

---

## 1. Planning and Scope

**Threat Intelligence Context:**
- [Why are we testing this? e.g., "Recent intelligence indicates Threat Actor X is actively using these techniques against our sector."]

**In-Scope Assets:**
- [List specific hostnames or IP addresses that will be targeted. e.g., `DC-PROD-01`, `APP-STG-04`]

**Out-of-Scope:**
- [Explicitly state what must NOT be touched. e.g., "No execution against the Payment Processing enclave. No actions that cause denial of service."]

**Techniques to Validate (ATT&CK Mapping):**
1. [T-Code] - [Technique Name]
2. [T-Code] - [Technique Name]

---

## 2. Execution Log

> *Copy this block for each specific technique or test variation executed.*

### Test Case: [e.g., T1558.003 - Kerberoasting via Rubeus]

**Expected Baseline:**
- [What do we think will happen? e.g., "SIEM should alert on high volume of EID 4769 RC4 requests from a workstation."]

**Execution Detail:**
- **Time Executed:** [HH:MM:SS]
- **Source System:** [Hostname / IP]
- **Target System:** [Hostname / IP]
- **Command / Tool Used:**
  ```text
  [Insert exact command line executed by Red]
  ```

**Observation (Blue):**
- **Telemetry Generated?** [Yes / No / Partial]
- **Alert Generated?** [Yes / No]
- **Rule ID Triggered:** [If Yes, insert Rule ID]
- **Analyst Triage Outcome:** [e.g., "Alert appeared in queue, correctly identified as High severity."]

**Gap Analysis:**
- [If it failed, why? e.g., "Telemetry generated, but rule did not fire because it was looking for EID 4768 instead of 4769."]

---

## 3. Real-Time Tuning

*Document any changes made to detection logic or configurations during the exercise.*

**Rule Updated:** [Rule ID]
**Logic Change:**
- [e.g., "Changed logic to alert on > 5 requests within 1 minute instead of 15 requests."]
**Retest Result:** [Pass / Fail]

---

## 4. Post-Exercise Action Items (Lessons Learned)

*Document structural fixes that could not be completed during the exercise.*

| Action Item | Owner | Target Date | Ticket ID |
|---|---|---|---|
| [e.g., Enable Sysmon EID 10 on all Domain Controllers] | [Name] | [Date] | [JIRA-1234] |
| [e.g., Write new SIEM rule for WMI lateral movement] | [Name] | [Date] | [JIRA-1235] |
| [e.g., Update IR Playbook for Kerberoasting to include password reset steps] | [Name] | [Date] | [JIRA-1236] |

---

## 5. Exercise Sign-Off

**Exercise Lead:** ____________________ Date: ________
**SOC Manager:** ____________________ Date: ________

---
**References:**
- [[Attack Detection Response Matrix]]
- [[Purple Team - Lessons Learned Model]]
