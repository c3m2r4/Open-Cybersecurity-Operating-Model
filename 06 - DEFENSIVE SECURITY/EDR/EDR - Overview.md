---
title: EDR - Overview
category: Security Operations
tags:
  - EDR
  - EndpointSecurity
  - Detection
  - IncidentResponse
date_created: 2026-08-24
status: MAINTAINED
---

# EDR — Overview

## Purpose

Endpoint Detection and Response (EDR) provides deep visibility into what is happening on an individual host (workstation or server). While SIEM aggregates logs from many sources, EDR focuses on continuous telemetry of endpoint behavior, providing both detection and the capability to isolate and investigate compromised systems.

> Antivirus stops known bad files. EDR detects malicious behavior, even when legitimate tools (living off the land) are used.

---

## Core Capabilities

| Capability | Description |
|---|---|
| **Telemetry Collection** | Continuous recording of process execution, network connections, file modifications, and registry changes |
| **Behavioral Detection** | Alerting on sequences of events (e.g., Word spawns PowerShell which connects to the internet) |
| **Threat Intelligence Matching** | Alerting if known malicious hashes, IPs, or domains are observed |
| **Isolation / Containment** | Blocking the endpoint's network access (except to the EDR console) to stop lateral movement |
| **Remote Investigation** | Allowing analysts to pull files, run commands, or dump memory remotely |
| **Response Actions** | Killing malicious processes, deleting files, or removing registry keys |

---

## Endpoint Telemetry: The Attacker vs Defender View

EDR bridges the gap between what an attacker does and what a defender sees.

### Process Execution

| Attacker Action | EDR Telemetry (What Defenders See) |
|---|---|
| Attacker runs a malicious macro in an Office document | `winword.exe` spawns `cmd.exe` or `powershell.exe` |
| Attacker uses PsExec for lateral movement | `services.exe` spawns `PSEXESVC.exe` |
| Attacker injects code into a legitimate process | `CreateRemoteThread` API call; abnormal memory allocation in target process |
| Attacker dumps LSASS memory for credentials | Untrusted process opens handle to `lsass.exe` with specific access rights |

### Network Connections

| Attacker Action | EDR Telemetry (What Defenders See) |
|---|---|
| C2 payload beacons to attacker infrastructure | Unknown/unsigned process makes outbound TCP/443 connection to rare domain |
| Attacker moves laterally via RDP | `mstsc.exe` connects to internal IP on port 3389 without standard IT admin context |
| Attacker exfiltrates data via curl | `curl.exe` connects to external cloud storage provider |

### File and Registry Activity

| Attacker Action | EDR Telemetry (What Defenders See) |
|---|---|
| Attacker establishes persistence via Run key | Process writes to `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |
| Ransomware encrypts files | High volume of rapid file rename/modify operations across user directories |
| Attacker drops a web shell | `w3wp.exe` (IIS) creates a `.aspx` file in the web root |

---

## Investigation Workflow in EDR

When an EDR alert fires, the SOC analyst uses the EDR console to investigate:

1. **Review the Process Tree:** What process generated the alert? What spawned that process? (e.g., If `explorer.exe` spawned the process, a user likely clicked it. If `w3wp.exe` spawned it, it's a web exploit).
2. **Review the Timeline:** What did the process do before and after the alert? Did it write files? Did it make network connections?
3. **Check for Lateral Movement:** Did the process connect to other internal endpoints via SMB or RPC?
4. **Isolate if Confirmed:** If the activity is clearly malicious, invoke host isolation to prevent lateral movement while investigation continues.

---

## Limitations of EDR

EDR is powerful, but it is not a silver bullet. Understand its blind spots:

| Limitation | Impact |
|---|---|
| **Coverage Gaps** | EDR cannot detect what happens on systems where it is not installed (e.g., unmanaged BYOD, IoT, some network appliances). |
| **Agent Tampering** | Advanced adversaries will attempt to blind, disable, or bypass the EDR agent. |
| **Kernel Level Access** | Rootkits operating below the level of the EDR agent may evade detection. |
| **Legitimate Tool Abuse** | Very difficult to distinguish a sysadmin legitimately using PowerShell from an attacker doing the same ("Living off the Land"). |
| **Cloud/Network Blindness** | EDR sees the endpoint. It does not see cloud control plane activity (AWS IAM) or network flow data. |

---

## Integration with SIEM

| Tool | Focus | Role in SOC |
|---|---|---|
| **EDR** | Endpoint behavior | Generates high-fidelity alerts; provides investigation console; executes containment |
| **SIEM** | Cross-domain correlation | Correlates EDR alerts with network, identity, and cloud logs to show the full attack picture |

> Example: EDR detects a suspicious PowerShell script. SIEM shows that the user account running that script authenticated from a new country 10 minutes prior via the VPN. EDR provides the endpoint detail; SIEM provides the context.

---

## Related Notes

- [[Control - Endpoint Detection and Response]] — EDR as a formal control
- [[Incident Response - Lifecycle]] — using EDR for containment
- [[SOC - Operating Model]] — how analysts use EDR in triage
- [[Mythic Lab - Project Bridge]] — testing EDR bypass and telemetry in the lab
