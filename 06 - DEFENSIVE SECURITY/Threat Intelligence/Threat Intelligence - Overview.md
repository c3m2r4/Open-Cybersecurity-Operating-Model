---
title: Threat Intelligence - Overview
category: Security Operations
tags:
  - ThreatIntelligence
  - SOC
  - Risk
date_created: 2026-08-24
status: MAINTAINED
---

# Threat Intelligence — Overview

## Purpose

Cyber Threat Intelligence (CTI) is analyzed information about the intent, capability, and opportunity of adversaries. It shifts an organization's defense from purely reactive (waiting for an alert) to proactive (anticipating the attack and preparing defenses).

> Raw data (a list of IP addresses) is not intelligence. Information (a report that a specific malware uses those IPs) is not intelligence. Intelligence is analyzed information that informs a decision (blocking those IPs and searching historical logs because that malware targets our specific industry).

---

## The Four Levels of Threat Intelligence

| Level | Audience | Focus | Output Example |
|---|---|---|---|
| **Strategic** | Board, C-Suite, CISO | Broad trends, financial impact, geopolitical shifts, major adversary goals. Informs security strategy and budget. | "Ransomware groups are increasingly targeting the manufacturing sector to extort via operational downtime." |
| **Operational** | Security Managers, IR Leads, Red Team | Details of specific adversary campaigns, TTPs (Tactics, Techniques, and Procedures), and targeting. Informs defensive priorities. | "Threat Actor X is using spear-phishing with malicious PDFs to gain initial access, followed by Cobalt Strike." |
| **Tactical** | SOC Analysts, Detection Engineers | Technical details on how threats operate. Informs detection rules and hunting hypotheses. | "The malware achieves persistence by modifying the `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` registry key." |
| **Technical** | SIEM, Firewalls, EDR (Automated) | Indicators of Compromise (IOCs). Highly volatile, short lifespan. Informs automated blocking and alerting. | "Block connections to IP `198.51.100.45` and hash `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`." |

---

## The Intelligence Lifecycle

Intelligence is not a feed you purchase; it is a continuous process.

1. **Direction:** Defining what intelligence the organization needs (Priority Intelligence Requirements - PIRs). *What are our crown jewels? Who would want them?*
2. **Collection:** Gathering raw data from internal sources (logs, incidents) and external sources (OSINT, paid feeds, ISACs).
3. **Processing:** Normalizing, structuring, and filtering the collected data.
4. **Analysis:** Human review to turn processed data into actionable insights. Contextualizing the threat.
5. **Dissemination:** Delivering the intelligence to the right audience in the right format (e.g., IOCs to the SIEM, a report to the CISO).
6. **Feedback:** Did this intelligence help the SOC? Were the IOCs high-fidelity? Adjust Direction based on feedback.

---

## Applying Threat Intelligence

CTI must integrate with other security domains to be valuable:

| Domain | How CTI is Applied |
|---|---|
| **Detection Engineering** | Building behavioral rules based on Tactical intel (TTPs) rather than just Technical intel (IOCs). |
| **Threat Hunting** | Formulating hypotheses based on Operational intel. ("Actor X targets our sector using technique Y. Let's search for technique Y.") |
| **Vulnerability Management** | Prioritizing patching based on whether a vulnerability is actively exploited in the wild (EPSS), rather than just its CVSS score. |
| **Red Teaming / Adversary Simulation** | Emulating specific threat actors based on Operational and Tactical intel to test organizational resilience. |
| **Risk Management** | Updating risk scenarios and likelihood assessments based on Strategic intel. |

---

## IOCs vs. TTPs

* **Indicators of Compromise (IOCs):** Hash values, IP addresses, domains. Easy for defenders to use (blocklists), but trivial for attackers to change. (See [[Detection Engineering - Quality Model]] - Pyramid of Pain).
* **Tactics, Techniques, and Procedures (TTPs):** How the attacker operates. (e.g., using WMI for lateral movement, dumping LSASS). Harder for defenders to detect, but much harder for attackers to change. CTI should heavily focus on TTPs.

---

## Related Notes

- [[Detection Engineering - Quality Model]] — Pyramid of Pain
- [[Red Team and Adversary Simulation - Overview]] — using CTI for simulation
- [[Risk-Based Offensive Security]] — using CTI to assess threat likelihood
- [[SOC - Operating Model]] — integrating CTI into SOC triage
