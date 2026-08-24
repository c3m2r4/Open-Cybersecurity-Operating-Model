---
title: Incident Response - Lifecycle
category: Security Operations
tags:
  - IncidentResponse
  - SOC
  - DFIR
date_created: 2026-08-24
status: MAINTAINED
---

# Incident Response — Lifecycle

## Purpose

Incident Response (IR) is the structured methodology for handling a security breach or cyberattack. The goal of IR is to manage the situation in a way that limits damage, reduces recovery time, and costs, while ensuring that the organization meets legal and regulatory requirements.

> A chaotic response often causes more business damage than the initial attack. The purpose of an IR plan is to impose order on a crisis.

---

## The 6-Phase IR Lifecycle (SANS/NIST Model)

```text
1. Preparation
     ↓
2. Identification (Detection & Analysis)
     ↓
3. Containment
     ↓
4. Eradication
     ↓
5. Recovery
     ↓
6. Lessons Learned (Post-Incident Review)
```

---

## Phase Details

### 1. Preparation

This phase occurs *before* an incident happens. You cannot build a response capability during a crisis.

| Key Activities |
|---|
| Establish the IR Team (roles, contact info, on-call schedules) |
| Develop and test playbooks for specific scenarios (e.g., Ransomware, BEC, Data Breach) |
| Ensure adequate logging, EDR deployment, and visibility |
| Establish out-of-band communication channels (if corporate email/Slack goes down) |
| Retain external incident response and legal counsel on retainer |
| Conduct tabletop exercises to practice response |

### 2. Identification (Detection & Analysis)

Determining whether an incident has occurred, its severity, and its scope.

| Key Activities |
|---|
| Triage alerts from SIEM, EDR, or user reports |
| Correlate indicators to understand the scope (Who is affected? What systems?) |
| Declare the incident and assign a severity level (Severity dictates escalation) |
| Establish a timeline of attacker activity |
| **Do NOT jump to eradication yet. Understand the full scope first, or the attacker will just use their backdoor to get back in.** |

### 3. Containment

Stopping the bleeding. Preventing the attacker from accessing more systems or exfiltrating more data.

| Containment Type | Example |
|---|---|
| **Short-Term** | Isolating an infected endpoint via EDR; disabling a compromised user account. |
| **Long-Term** | Taking a critical application offline; severing external network connections; rebuilding a segment. |

> Containment decisions involve business risk. Shutting down the entire network contains the threat but causes a massive business outage. The IR Lead must coordinate with business stakeholders.

### 4. Eradication

Removing the threat from the environment entirely.

| Key Activities |
|---|
| Delete malware, web shells, and persistence mechanisms |
| Rebuild infected systems from clean images (often preferred over manual cleaning) |
| Force password resets for compromised accounts |
| Remove malicious inbox rules or forwarding rules |
| Patch the vulnerability that allowed the initial access |

### 5. Recovery

Restoring systems to normal business operations.

| Key Activities |
|---|
| Restore data from clean, verified backups |
| Bring isolated networks or applications back online |
| Monitor the environment closely for signs of re-infection (attackers often try to return immediately) |
| Validate that systems are functioning correctly |

### 6. Lessons Learned (Post-Incident Review)

The most important and most frequently skipped phase.

| Key Activities |
|---|
| Conduct a blameless post-mortem within two weeks of the incident |
| Ask: What went well? What failed? What visibility did we lack? |
| Document specific improvement actions (e.g., "Deploy MFA to legacy app," "Create detection rule for XYZ") |
| Assign owners and deadlines to improvement actions |

See: [[Purple Team - Lessons Learned Model]]

---

## Regulatory and Legal Considerations

> VERIFY: Legal and regulatory requirements vary strictly by jurisdiction and industry.

Incident response is not purely technical. It involves legal risk, breach notification laws, and contractual obligations.

- **Preservation of Evidence:** Ensure logs and forensic artifacts are not destroyed during containment. See [[Digital Forensics - Overview]].
- **Notification Windows:** Many regulations (e.g., GDPR) require notification to authorities within 72 hours of becoming aware of a breach.
- **Legal Privilege:** In severe incidents, external IR firms should often be engaged through external legal counsel to maintain attorney-client privilege over the investigation findings.

---

## Related Notes

- [[Incident Response - Roles and Escalation]] — who does what during an incident
- [[SOC - Operating Model]] — how alerts escalate into incidents
- [[Digital Forensics - Overview]] — evidence handling during IR
- [[Control - Incident Response Process]] — the formal control
