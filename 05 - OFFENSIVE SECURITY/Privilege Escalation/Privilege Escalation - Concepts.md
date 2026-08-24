---
title: Privilege Escalation - Concepts
category: Security Domain
tags:
  - PrivilegeEscalation
  - OffensiveSecurity
  - AttackPath
date_created: 2026-08-24
status: MAINTAINED
---

# Privilege Escalation — Concepts

## What Is Privilege Escalation?

Privilege escalation is the process of gaining a higher level of access than was initially obtained. It is a necessary step in most attack paths: initial access is rarely obtained with the highest available privilege, so attackers escalate to reach their objectives.

```text
Initial Access (low privilege)
  → Privilege Escalation (local admin, service account, domain admin)
  → Lateral Movement to higher-value systems
  → Impact (data access, persistence, disruption)
```

---

## Types of Privilege Escalation

### Local Privilege Escalation

Gaining elevated privilege on the system where access was obtained.

| Category | Description |
|---|---|
| **Kernel / OS exploits** | Exploiting unpatched operating system vulnerabilities to gain system/root privilege |
| **Service misconfigurations** | Writable service binaries; weak service permissions; unquoted service paths |
| **Scheduled task abuse** | Tasks running as SYSTEM with writable scripts or executables |
| **Registry misconfigurations** | Writable registry keys used by elevated processes |
| **DLL hijacking** | Placing a malicious DLL in a path searched by a privileged process |
| **Token manipulation** | Stealing or impersonating tokens from privileged processes |
| **AlwaysInstallElevated** | MSI installs always run as SYSTEM when registry keys are set |
| **Credentials in files or registry** | Stored credentials providing access to higher-privilege accounts |

---

### Domain Privilege Escalation

Gaining elevated privilege within an Active Directory domain from a lower-privilege domain account.

| Category | Description |
|---|---|
| **ACL abuse** | Write permissions on privileged objects enable password reset or group membership modification |
| **Kerberos delegation abuse** | Unconstrained or misconfigured constrained delegation enables impersonation of privileged accounts |
| **ADCS abuse** | Misconfigured certificate templates enable authentication as privileged accounts |
| **Group Policy abuse** | Write permissions on GPO linked to privileged OU |
| **AS-REP Roasting** | Accounts with pre-auth disabled; offline hash cracking |
| **Kerberoasting** | Service accounts with weak passwords; offline hash cracking |
| **Pass-the-Hash / Pass-the-Ticket** | Using captured credentials or tickets to authenticate as privileged accounts |
| **DCSync** | Replication permissions enabling credential extraction from domain controllers |

See: [[Active Directory Security - Overview]], [[Active Directory - Attack Path Analysis]]

---

### Cloud Privilege Escalation

Gaining elevated privilege in a cloud environment from a lower-privilege identity.

| Category | Description |
|---|---|
| **IAM role privilege escalation** | Misconfigured IAM policies allow attaching more permissive roles |
| **Instance metadata service (IMDS)** | EC2/compute instance credentials obtainable via SSRF or local access |
| **Function / Lambda privilege escalation** | Serverless functions with broad permissions execute attacker-controlled code |
| **Managed identity abuse** | Azure / GCP managed identities with broad permissions accessed from compromised resources |

See: [[Cloud Security Testing - Overview]]

---

## Business Risk

Privilege escalation dramatically amplifies the impact of initial access:

| Escalation Level | Business Risk |
|---|---|
| Local admin on one workstation | Access to local credentials; lateral movement platform |
| Local admin on server | Server data access; credentials in memory; broader lateral movement |
| Domain admin | Complete domain compromise; all systems, data, and accounts accessible |
| Cloud global admin | Complete cloud environment compromise; all data and services |
| Active Directory Certificate Authority | Persistent authentication as any identity; long-term dwell |

---

## Detection Expectations

What defenders should look for:

| Technique Category | Expected Telemetry |
|---|---|
| Local service exploitation | Unexpected SYSTEM process creation; service binary modification |
| ACL modification | 5136 event (AD object modified) — ACE added to privileged object |
| Kerberoasting | 4769 events with RC4 encryption type from workstation accounts |
| ADCS abuse | Certificate enrollment events from unexpected accounts or templates |
| Token manipulation | Unusual process ownership; process injection indicators |
| Privilege use | 4672 (special privilege assigned) from unexpected accounts or systems |

See: [[Active Directory - Detection and Hardening]], [[EDR - Overview]]

---

## Hardening

| Area | Hardening Action |
|---|---|
| Patch management | Keep OS and software patched to reduce kernel/service exploit surface |
| Least privilege | Local admin rights should be limited to authorized accounts only |
| Service account hygiene | Service accounts should have only permissions required; use gMSA |
| ACL governance | Regular audit of write permissions on privileged AD objects |
| ADCS hardening | Review and restrict certificate templates |
| Tiered administration | Admin credentials should not be cached on lower-tier systems |
| EDR | Monitor for privilege escalation indicators (token manipulation, process injection) |

See: [[Control - Privileged Access Management]], [[Control - Least Privilege]]

---

## Lab Bridge

Privilege escalation in the Active Directory context is validated in the GOAD lab:

| Note | Purpose |
|---|---|
| [[GOAD Project - Risk and Control Bridge]] | Maps GOAD privilege escalation paths to business risk |
| [[GOAD - Command Execution Matrix]] | Authorized technical procedures |
| [[Mythic Lab - Project Bridge]] | Post-exploitation and token manipulation in authorized lab |

---

## Related Notes

- [[Active Directory Security - Overview]] — domain privilege escalation context
- [[Lateral Movement - Concepts]] — what comes after privilege escalation
- [[Risk-Based Offensive Security]] — how to rate privilege escalation findings
- [[Offensive Security Methodology]] — Phase 10 (Controlled Exploitation), Phase 11 (Impact Assessment)
