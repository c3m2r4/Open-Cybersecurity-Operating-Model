---
title: Lateral Movement - Concepts
category: Security Domain
tags:
  - LateralMovement
  - OffensiveSecurity
  - NetworkSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Lateral Movement — Concepts

## What Is Lateral Movement?

Lateral movement is the process of using obtained access or credentials to move from one system to another within an environment. It amplifies initial compromise — an attacker who starts with access to one low-value workstation may move laterally to servers, domain controllers, or data repositories.

```text
Initial Compromise (one system)
  → Credential Acquisition (local, cached, or domain credentials)
  → Lateral Movement (authenticate to other systems)
  → Higher-Value Target (server, DC, Crown Jewels)
```

---

## Why Lateral Movement Matters

Most high-impact breaches do not end at the initially compromised system. The initial breach is a foothold; lateral movement is how adversaries reach valuable data, systems, or domain control. Defending against lateral movement means:

- Reducing the reuse of credentials across systems
- Enforcing network segmentation
- Detecting authentication from unexpected sources
- Detecting movement between systems outside normal patterns

---

## Technique Categories

### Credential-Based Movement

| Technique | Description | Weakness Exploited |
|---|---|---|
| **Pass-the-Hash (PtH)** | Using the NTLM hash of a password to authenticate without knowing the plaintext password | NTLM authentication accepts hashes directly |
| **Pass-the-Ticket (PtT)** | Using a stolen Kerberos ticket to authenticate to services | Valid tickets are accepted regardless of how they were obtained |
| **Credential reuse** | Using a password obtained from one system to authenticate to others | Password reuse across systems; local admin account reuse |
| **Over-pass-the-Hash (Overpass-the-Hash)** | Using an NTLM hash to request a Kerberos TGT | NTLM hash enables Kerberos ticket generation |

---

### Protocol-Based Movement

| Protocol | Lateral Movement Use | Detection Signal |
|---|---|---|
| **SMB** | Accessing file shares; remote execution via SMB (PsExec-style) | Network share access events; remote service creation |
| **WMI** | Remote command execution via Windows Management Instrumentation | WMI query events; unusual process creation from WMI |
| **RDP** | Interactive remote desktop access | 4624 logon type 10; RDP connection events |
| **WinRM / PowerShell Remoting** | Remote PowerShell execution | PowerShell remote connection events; 4624 logon type 3 |
| **DCOM** | Distributed COM object remote execution | Process creation from unusual parent; DCOM connection events |
| **SSH** | Linux/Unix lateral movement | SSH authentication events from unusual source |

---

### Token and Impersonation

| Technique | Description |
|---|---|
| **Token impersonation** | Stealing or duplicating the security token of a logged-in user to act with their privileges |
| **Named pipe impersonation** | Causing a privileged process to connect to an attacker-controlled named pipe; impersonating the resulting token |

---

## Business Risk of Lateral Movement

| Movement Target | Business Risk |
|---|---|
| Additional workstations | Broader credential harvesting; wider persistence |
| File servers | Access to organizational data; intellectual property; sensitive documents |
| Database servers | Direct database access; bulk data theft |
| Application servers | Application-level access; backend business system access |
| Domain Controllers | Domain compromise — all credentials, all systems, all trust |
| Backup servers | Ransomware target; removal of recovery capability |

The **blast radius** of a compromise is determined by how far an attacker can move laterally. Network segmentation directly limits this.

---

## Segmentation and Lateral Movement

Network segmentation is the primary architectural control against lateral movement:

| Segmentation Design | Effect on Lateral Movement |
|---|---|
| Flat network (all systems reachable) | Any compromised host can attempt to reach any other host |
| VLAN segmentation (servers separated from workstations) | Reduces direct workstation-to-server movement |
| Micro-segmentation (east-west traffic controlled) | Limits lateral movement significantly; requires firewall rules per workstation |
| Tiered administration (Tier 0/1/2 enforced) | Tier 0 credentials cannot authenticate to Tier 2 systems |

See: [[Control - Network Segmentation]]

---

## Detection Expectations

| Technique | Detection Signal |
|---|---|
| Pass-the-Hash | 4624 logon type 3 with NTLM auth package from unexpected source |
| Pass-the-Ticket | 4768/4769 with unusual source IP or forged ticket indicators |
| SMB lateral movement | 4624 type 3 logon from workstation to server; 5145 share access events |
| RDP movement | 4624 type 10 logon; terminal services logon events |
| WMI remote execution | WMI consumer creation events; process creation from WMI parent |
| PowerShell Remoting | 4624 type 3 from unusual source; PowerShell event log entries |
| Credential reuse | Logon from account on system it has no business reason to access |

Route authentication telemetry to [[SIEM - Overview]] for baselining and anomaly detection.

---

## Hardening Against Lateral Movement

| Control | Effect |
|---|---|
| **Credential Guard** | Prevents LSASS credential extraction for PtH |
| **Local Admin Password Solution (LAPS)** | Unique local administrator password per workstation — prevents lateral movement via shared local admin credential |
| **Network Segmentation** | Limits which systems can communicate with which |
| **Tiered Administration** | Prevents high-value credentials from being cached on low-trust systems |
| **Disable NTLM where possible** | Reduces PtH and relay attack surface |
| **MFA for lateral movement protocols** | MFA for RDP and remote management reduces credential-only movement |
| **SMB signing** | Prevents SMB relay attacks |
| **EDR** | Detects credential access, process injection, and remote execution indicators |

See: [[Control - Network Segmentation]], [[Control - Privileged Access Management]], [[EDR - Overview]]

---

## Lab Bridge

Lateral movement in the Active Directory context is validated in the GOAD lab, and C2-enabled lateral movement concepts are explored in the Mythic lab:

| Note | Purpose |
|---|---|
| [[GOAD Project - Risk and Control Bridge]] | AD lateral movement paths and business risk |
| [[Mythic Lab - Project Bridge]] | C2 and post-exploitation lateral movement concepts |
| [[GOAD - Active Directory - Inside the Domain Workflow]] | Post-compromise navigation workflow |

---

## Related Notes

- [[Privilege Escalation - Concepts]] — privilege escalation often precedes lateral movement
- [[Active Directory Security - Overview]] — AD context for credential-based movement
- [[Active Directory - Detection and Hardening]] — detection for AD lateral movement
- [[Control - Network Segmentation]] — segmentation control
