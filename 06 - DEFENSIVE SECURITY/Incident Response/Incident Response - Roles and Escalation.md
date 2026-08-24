---
title: Incident Response - Roles and Escalation
category: Security Operations
tags:
  - IncidentResponse
  - SOC
  - Governance
date_created: 2026-08-24
status: MAINTAINED
---

# Incident Response — Roles and Escalation

## Purpose

During a major security incident, confusion over who is in charge, who is authorized to make decisions, and who communicates with external parties causes delays that attackers exploit. Clearly defined roles and escalation thresholds are critical to effective incident response.

---

## Core Incident Response Roles

| Role | Responsibilities |
|---|---|
| **Incident Commander (IC)** | The single point of authority during the incident. Manages the overall response, coordinates teams, and makes tactical decisions. Not performing hands-on technical work. |
| **Lead Investigator** | Directs the technical investigation, directs analysts, formulates the timeline, and determines the scope of the compromise. |
| **SOC Analysts / Technical Responders** | Execute containment actions, run queries, analyze logs, and reverse engineer malware. |
| **Scribe / Documentation Lead** | Maintains the official timeline, notes decisions, and tracks action items. (Crucial for post-incident review and legal defense). |
| **Communications Lead** | Manages updates to executives, employees, and customers. Ensures technical details are translated accurately for non-technical audiences. |
| **Legal Counsel** | Advises on regulatory obligations, breach notification requirements, and liability. Often engages external IR firms to establish privilege. |
| **Business Stakeholder / Executive Sponsor** | Approves high-impact containment decisions (e.g., "Take the main revenue-generating app offline"). |

> **Key Rule:** The Incident Commander runs the incident. Executives must not bypass the IC to give direct orders to technical analysts.

---

## Severity Levels and Escalation Matrix

Incidents must be classified by severity to determine the level of response and executive notification required.

> VERIFY: These definitions should be customized to the specific organization's risk tolerance and business model.

### Severity 1 (Critical / Major Incident)
* **Definition:** Widespread disruption of critical business processes; confirmed mass data exfiltration; active ransomware across the enterprise; loss of domain control.
* **Escalation:** Immediate notification to C-Suite, Legal, and PR. Activation of full crisis management team. External IR firm likely engaged.
* **Update Cadence:** Hourly briefings to executive sponsor.

### Severity 2 (High)
* **Definition:** Compromise of a critical system or executive account; localized data exposure; active lateral movement detected. Business impact is significant but contained to specific areas.
* **Escalation:** Immediate notification to CISO/Security Director and IT Leadership. Legal briefed.
* **Update Cadence:** Every 4-6 hours.

### Severity 3 (Medium)
* **Definition:** Compromise of a non-critical system; localized malware infection effectively contained; successful phishing of standard user (contained).
* **Escalation:** Managed within the SOC. SOC Manager notified.
* **Update Cadence:** Daily update or upon closure.

### Severity 4 (Low / Triage)
* **Definition:** Unsuccessful attacks; policy violations; adware; blocked malware.
* **Escalation:** Handled by Tier 1/2 analysts per standard playbooks. No executive notification.
* **Update Cadence:** Aggregated in weekly/monthly metric reports.

---

## The Escalation Process

1. **Trigger:** A SOC analyst identifies activity matching the criteria for a Sev-2 or Sev-1 incident.
2. **Declaration:** The analyst escalates to the SOC Manager or designated IR Lead. The IR Lead formally *declares* the incident and assigns the severity.
3. **Mobilization:** The IC pages the required roles based on the severity matrix.
4. **War Room:** A dedicated, secure communication channel (e.g., specific Slack channel, Teams bridge) is established. Out-of-band comms (e.g., Signal, separate tenant) are used if the primary network is compromised.

---

## Communication Disciplines

* **Need to Know:** Do not broadcast incident details to the entire company. Attackers may be monitoring internal communications.
* **Facts vs. Assumptions:** State clearly what is confirmed fact vs. what is a working hypothesis. (e.g., "We *hypothesize* the attacker entered via the VPN, but we have *confirmed* they accessed the database.")
* **Out-of-Band Comms:** If O365/Exchange is compromised, emailing the IR plan to the team is handing it to the attacker. Use established out-of-band methods.

---

## Related Notes

- [[Incident Response - Lifecycle]] — the phased approach to handling the incident
- [[SOC - Operating Model]] — how alerts become incidents
