---
title: Detection Engineering - Lifecycle
category: Security Operations
tags:
  - DetectionEngineering
  - SIEM
  - SOC
date_created: 2026-08-24
status: MAINTAINED
---

# Detection Engineering — Lifecycle

## Purpose

Detection engineering is the discipline of creating, testing, and maintaining logic that identifies malicious activity. It treats detection as code. The lifecycle ensures that rules are built based on threat intelligence, tested before deployment, and tuned to maintain a high signal-to-noise ratio.

> A detection rule deployed without testing is a liability. It will either bury the SOC in false positives or fail silently when a real attack occurs.

---

## The Detection Lifecycle

```text
1. Threat Identification
     ↓
2. Behavior Analysis
     ↓
3. Telemetry Assessment
     ↓
4. Logic Development
     ↓
5. Testing & Validation
     ↓
6. Tuning
     ↓
7. Deployment
     ↓
8. Monitoring & Maintenance
```

---

## Phase Details

### 1. Threat Identification

| Action | Detail |
|---|---|
| **Input** | Threat intelligence; offensive security findings; red team reports; new vulnerabilities |
| **Output** | A prioritized list of techniques that need detection coverage |
| **Example** | "We need to detect attackers dumping LSASS memory for credentials, as reported in the latest pentest." |

### 2. Behavior Analysis

| Action | Detail |
|---|---|
| **Input** | Threat intelligence on the technique; MITRE ATT&CK |
| **Output** | An understanding of the underlying behavior, independent of the specific tool |
| **Example** | "Tools like Mimikatz, ProcDump, and Task Manager all open a handle to `lsass.exe` with specific access rights to read memory." |

### 3. Telemetry Assessment

| Action | Detail |
|---|---|
| **Input** | Behavioral understanding; log source inventory |
| **Output** | Confirmation of which log sources provide the necessary visibility |
| **Example** | "To see process access handles, we need Sysmon Event ID 10 or Windows Event ID 4656. Do we collect this from Domain Controllers? Yes." |

### 4. Logic Development

| Action | Detail |
|---|---|
| **Input** | Telemetry assessment; SIEM query language |
| **Output** | Draft detection logic and documented use case |
| **Example** | `TargetImage="*lsass.exe" AND GrantedAccess="0x1010" OR GrantedAccess="0x1410"` |

See: [[SIEM - Detection Use Cases]]

### 5. Testing & Validation

| Action | Detail |
|---|---|
| **Input** | Draft logic; lab environment; purple team |
| **Output** | Proof that the rule fires on true positive activity |
| **Example** | Execute Mimikatz in the lab. Does the alert fire? Execute a different tool (ProcDump). Does it still fire? |

See: [[Detection Validation - Overview]]

### 6. Tuning

| Action | Detail |
|---|---|
| **Input** | Tested logic; historical production data |
| **Output** | Exclusions for known legitimate activity |
| **Example** | Run the query against 30 days of production data. Oh, the backup agent opens handles to LSASS daily. Add an exclusion for `SourceImage="*backup_agent.exe"`. |

### 7. Deployment

| Action | Detail |
|---|---|
| **Input** | Tuned logic; SOC response playbook |
| **Output** | Active alert generating tickets for the SOC |
| **Example** | Deploy rule to production. Ensure the alert contains clear instructions for the Tier 1 analyst on how to investigate. |

### 8. Monitoring & Maintenance

| Action | Detail |
|---|---|
| **Input** | Active alert; SOC feedback |
| **Output** | Ongoing tuning; deprecation of obsolete rules |
| **Example** | The SOC reports this rule generates 15 false positives a week due to a new software deployment. The detection engineer updates the logic to exclude the new software. |

---

## Detection as Code

Mature detection engineering teams manage rules like software development:
- Rules are stored in version control (Git)
- Changes are reviewed via Pull Requests
- CI/CD pipelines validate syntax and deploy to the SIEM
- Automated tests run periodically to ensure rules haven't broken due to log schema changes

---

## Related Notes

- [[Detection Engineering - Quality Model]] — how to measure rule quality
- [[SIEM - Detection Use Cases]] — how to document rules
- [[Purple Team - Lifecycle]] — collaborative testing
- [[Threat Hunting - Overview]] — feeding hunts back into detection engineering
