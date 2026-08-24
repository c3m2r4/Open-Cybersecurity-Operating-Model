---
title: Active Directory Security - Overview
category: Security Domain
tags:
  - ActiveDirectory
  - IdentitySecurity
  - OffensiveSecurity
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Active Directory Security — Overview

## Executive Summary

Active Directory is the identity and access foundation of most Windows-based enterprise environments. It manages authentication, authorization, group policy, and trust relationships across the organization. Because of its central role, Active Directory is a high-value target — compromise of the directory often means compromise of every system that trusts it.

This note covers identity, authentication, authorization, attack paths, detection, and hardening concepts. Detailed laboratory procedures are in [[GOAD Project - Risk and Control Bridge]] and [[GOAD - Vault Index]].

---

## Core Concepts

### Identity and Directory

| Concept | Description |
|---|---|
| **Active Directory Domain Services (AD DS)** | The directory service that stores and manages identity objects: users, computers, groups, and organizational units |
| **Domain** | A logical grouping of directory objects managed by a common domain controller |
| **Domain Controller (DC)** | The server that runs AD DS, manages authentication, and replicates the directory database |
| **Forest** | A collection of domains that share a common schema, global catalog, and trust relationships |
| **Organizational Unit (OU)** | A container used to organize objects within a domain; target for Group Policy Object application |
| **Group Policy Object (GPO)** | Configuration enforced by domain controllers across computers and users in scope |

### Authentication Protocols

| Protocol | Description | Security Notes |
|---|---|---|
| **Kerberos** | Ticket-based authentication protocol; default in modern AD environments | Tickets can be forged (Golden Ticket, Silver Ticket) or stolen (Pass-the-Ticket) if key material is compromised |
| **NTLM** | Challenge-response authentication protocol; legacy | Susceptible to relay attacks; should be restricted where not required |
| **LDAP / LDAPS** | Directory query protocol; used for enumeration and authentication | LDAP without TLS exposes directory queries in cleartext |

### Authorization

| Concept | Description |
|---|---|
| **Security Group** | Group used to assign permissions to resources |
| **Access Control List (ACL)** | Defines who has what permissions on a directory object; misconfigurations are a primary attack path |
| **Delegation** | Allowing a user, computer, or service to act on behalf of another — Kerberos constrained, unconstrained, and resource-based constrained delegation have different risk profiles |
| **AdminSDHolder** | Protects privileged group members; AdminSDHolder ACL propagated to protected objects |
| **Protected Users** | A security group that disables legacy authentication and credential caching for members |

### Privileged Groups

These groups grant significant privilege and are high-value targets:

| Group | Privilege Level |
|---|---|
| Domain Admins | Full control over the domain |
| Enterprise Admins | Full control over the forest |
| Schema Admins | Can modify the Active Directory schema |
| Backup Operators | Can read all files; used in some backup-related privilege escalation paths |
| Account Operators | Can manage user and group objects in certain containers |
| Server Operators | Local admin on domain controllers |
| Administrators (local on DC) | Elevated on domain controllers |

> Membership in these groups should be limited, regularly reviewed, and monitored.

### Active Directory Certificate Services (ADCS)

| Concept | Description |
|---|---|
| **ADCS** | Microsoft's PKI implementation integrated with Active Directory |
| **Certificate Templates** | Define what certificates can be issued and to whom |
| **Certificate Authority (CA)** | Issues certificates; Enterprise CAs are joined to the domain and can issue certificates for AD authentication |
| **Risk** | Misconfigured certificate templates can allow privilege escalation, authentication as arbitrary users, or persistence (ADCS is covered in lab GOAD) |

---

## Attack Path Concepts

### How Attack Paths Form

Active Directory attacks typically follow a pattern:

```text
Initial Access
  → Credential Acquisition (local credentials, cached credentials, service account)
  → Enumeration (users, groups, ACLs, GPOs, delegation, trusts)
  → Lateral Movement (using acquired credentials to access other systems)
  → Privilege Escalation (exploiting ACL misconfiguration, delegation, ADCS, group membership)
  → Domain Control (obtaining domain admin or equivalent privilege)
```

No single step requires sophisticated exploitation. Many paths rely entirely on misconfiguration and excessive privilege.

### Common Weakness Categories

| Category | Examples |
|---|---|
| **Excessive Privilege** | Service accounts with Domain Admin; users in privileged groups without business justification |
| **Delegation Misconfiguration** | Unconstrained delegation on non-DC systems; misconfigured resource-based constrained delegation |
| **ACL Abuse** | User has GenericWrite on a privileged group; user has WriteDACL on a computer object |
| **Credential Exposure** | Passwords in SYSVOL, GPP files, scripts, or description fields; LSASS credential dumping paths |
| **ADCS Misconfiguration** | ESC1–ESC8 — misconfigured certificate templates, CA permissions, or issuance policies |
| **Kerberoasting** | Service accounts with weak passwords and SPNs set are vulnerable to offline password cracking |
| **AS-REP Roasting** | Accounts with Kerberos pre-authentication disabled allow encrypted hash retrieval without credentials |
| **Trust Abuse** | Cross-forest trusts with SID filtering disabled or insufficient trust restrictions |

### Business Risk Connection

| Attack Path | Business Risk |
|---|---|
| Domain Admin compromise | Complete loss of control over the domain; all systems, accounts, and data accessible |
| Service account credential theft | Lateral movement to all systems where the service account is used; potential data access |
| ADCS privilege escalation | Persistent authentication as any user; potential for long-term, hard-to-detect access |
| GPO modification | Organization-wide configuration change; potential for malware distribution or policy removal |
| Trust abuse | Cross-domain or cross-forest lateral movement; expansion of compromise beyond initial domain |

See: [[Active Directory - Attack Path Analysis]] for detailed analysis, [[Active Directory - Detection and Hardening]] for defensive guidance.

---

## Detection Expectations

What defenders should expect to see in telemetry when AD attack techniques are used:

| Technique | Expected Telemetry Source | Key Event / Indicator |
|---|---|---|
| Kerberoasting | Domain Controller Windows Event Log | 4769 (Kerberos Service Ticket request) with encryption type 0x17 (RC4) for service accounts |
| AS-REP Roasting | Domain Controller Windows Event Log | 4768 (Kerberos Authentication Ticket) with pre-auth type 0 |
| Pass-the-Hash (NTLM) | Domain Controller / SIEM | 4624 logon type 3 with NTLM auth from unusual source |
| Pass-the-Ticket | Domain Controller Windows Event Log | 4768 / 4769 with unusual source IP or forged ticket indicators |
| DCSync (credential dump via replication) | Domain Controller Windows Event Log | 4662 — replication permissions used by non-DC account |
| LDAP Enumeration | Domain Controller / SIEM | High-volume LDAP queries from workstation; unusual account querying AD attributes |
| GPO Modification | Domain Controller Windows Event Log | 5136 (Directory Service Object modified) on GPO objects |
| ADCS Certificate Enrollment | ADCS / Domain Controller Event Log | Certificate enrollment from unusual account or template |

> These are general indicators. Actual detection logic should be built and tuned in your specific environment. Do not implement detections based on this note alone without validation.

See: [[Detection Engineering - Lifecycle]], [[SIEM - Detection Use Cases]]

---

## Hardening Concepts

### Tiered Administration Model

| Tier | Scope | Purpose |
|---|---|---|
| Tier 0 | Domain Controllers, ADCS, identity management | Most privileged; most restricted |
| Tier 1 | Servers and applications | Server administrators |
| Tier 2 | Workstations and end-user devices | End-user support |

> Administrative accounts should only authenticate at the same or lower tier. Tier 0 credentials must not log into Tier 2 workstations.

### Key Hardening Controls

| Area | Hardening Action |
|---|---|
| Kerberos | Enable AES encryption; disable RC4 where possible; implement SID filtering on trusts |
| NTLM | Restrict NTLM authentication; monitor NTLM usage; block NTLM for domain admin accounts |
| Delegation | Audit and remove unconstrained delegation from non-DC systems; review RBCD configurations |
| Privileged Groups | Reduce membership; implement PAW; review regularly |
| Protected Users | Add sensitive accounts to Protected Users group |
| ADCS | Review certificate templates; restrict enrollment permissions; enable audit logging |
| Service Accounts | Use gMSA (Group Managed Service Accounts); rotate passwords regularly; monitor for SPN changes |
| LSASS | Enable Credential Guard; configure LSASS protection |
| SYSVOL / Scripts | Audit GPP files for stored credentials; review SYSVOL for sensitive data |

See also: [[Control - Multi-Factor Authentication]], [[Control - Privileged Access Management]]

---

## Lab Bridge

All Active Directory technical laboratory procedures are in the GOAD environment:

| GOAD Note | Purpose |
|---|---|
| [[GOAD - Vault Index]] | Navigation for all GOAD notes |
| [[GOAD Project - Risk and Control Bridge]] | Risk and control chain for AD validation |
| [[GOAD - Command Execution Matrix]] | Authorized lab procedure map |
| [[GOAD - Active Directory - Inside the Domain Workflow]] | Post-compromise navigation |

**Do not duplicate GOAD procedures here.** Use this note for concepts and risk framing; use GOAD notes for technical execution in the authorized lab.

---

## Related Notes

- [[Active Directory - Attack Path Analysis]] — how attack paths form and their business impact
- [[Active Directory - Detection and Hardening]] — defensive guidance in detail
- [[Privilege Escalation - Concepts]] — paths from low to high privilege
- [[Lateral Movement - Concepts]] — movement between systems
- [[Control - Privileged Access Management]] — PAM control in Batch 2
- [[Threat Intelligence - Overview]] — threat actor interest in AD environments
