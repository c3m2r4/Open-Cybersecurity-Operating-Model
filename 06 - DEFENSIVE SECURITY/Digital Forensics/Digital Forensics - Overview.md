---
title: Digital Forensics - Overview
category: Security Operations
tags:
  - DigitalForensics
  - DFIR
  - Evidence
date_created: 2026-08-24
status: MAINTAINED
---

# Digital Forensics — Overview

## Purpose

Digital Forensics is the scientific, methodical collection, preservation, and analysis of digital evidence. While incident response focuses on stopping the bleeding and restoring business operations, digital forensics focuses on establishing exactly what happened, when, and by whom, in a manner that can withstand legal scrutiny.

> If you do not preserve evidence correctly during containment, you destroy the forensic record required for post-incident legal defense or regulatory reporting.

---

## Core Principles

1. **Preservation of Evidence:** Do not alter the original evidence. Always work from a bit-for-bit copy (an image).
2. **Chain of Custody:** Document exactly who collected the evidence, where it was stored, who accessed it, and when. A broken chain of custody makes evidence inadmissible in court.
3. **Order of Volatility:** Collect evidence starting with the most volatile data (RAM, network connections) before moving to less volatile data (hard drives, backup tapes).
4. **Integrity:** Use cryptographic hashes (SHA-256) to prove that the image analyzed is exactly identical to the original evidence collected.

---

## The Order of Volatility (RFC 3227)

When responding to a compromised system, collect data in this order, as the act of collection alters the system state:

1. Registers, cache
2. Routing table, ARP cache, process table, kernel statistics, memory (RAM)
3. Temporary file systems
4. Disk (HDD/SSD)
5. Remote logging and monitoring data that is relevant to the system
6. Physical configuration, network topology
7. Archival media (backups)

> **Common IR Mistake:** "Reboot the server to see if it fixes the issue." Rebooting destroys RAM, drops network connections, kills running processes, and may trigger malware to delete itself or encrypt the drive.

---

## Key Forensic Artifacts (Windows)

| Artifact | What it Reveals |
|---|---|
| **RAM (Memory)** | Running processes, injected code, encryption keys, cleartext passwords, active network connections. |
| **MFT (Master File Table)** | File creation, modification, and access times (MAC times). Can show evidence of file deletion or timestomping. |
| **Windows Event Logs** | Authentication, service creation, privilege use. (Must be correlated with external SIEM). |
| **Registry** | Persistence mechanisms (Run keys), executed programs, connected USB devices. |
| **Prefetch / Amcache** | Evidence of application execution, even if the application has since been deleted. |
| **Browser History / Cache** | Web-based compromise, downloaded files, attacker web searches. |

---

## Acquisition vs. Analysis

### 1. Acquisition
The process of capturing the evidence.
- **Dead Box:** The system is powered off. The drive is removed and imaged using a write-blocker to prevent alteration.
- **Live Acquisition:** The system is running. Volatile memory (RAM) is captured, and the disk is imaged over the network or to a USB drive using specialized tools. (Necessary for full-disk encryption scenarios).
- **Triage Collection:** Pulling specific, high-value artifacts (Event Logs, Registry, MFT) rather than imaging a 2TB drive. Often handled by EDR or tools like KAPE.

### 2. Analysis
The process of interpreting the captured data.
- Building a timeline of events (Super Timeline).
- Identifying Indicators of Compromise (IOCs).
- Reconstructing attacker activity (Initial Access → Execution → Lateral Movement → Exfiltration).

---

## Digital Forensics in the Cloud

Cloud forensics differs significantly from traditional on-premises forensics:
- You cannot physically seize a server or attach a physical write-blocker.
- Acquisition relies on cloud provider APIs (e.g., snapshotting an EBS volume in AWS).
- Memory capture is difficult or impossible in serverless/PaaS environments.
- High reliance on Cloud Control Plane logs (CloudTrail, Azure Activity).

---

## Related Notes

- [[Incident Response - Lifecycle]] — where forensics fits in the response
- [[EDR - Overview]] — EDR tools often automate triage collection
- [[Operation Cypher-Knife - Scope]] — managing evidence in authorized operations
