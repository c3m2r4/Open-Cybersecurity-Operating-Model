---
title: Active Directory - Detection and Hardening
category: Security Domain
tags:
  - ActiveDirectory
  - Detection
  - Hardening
  - DefensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Active Directory — Detection and Hardening

## Purpose

This note covers what defenders should configure to detect Active Directory attacks and what hardening reduces attack surface. It connects attack path concepts from [[Active Directory - Attack Path Analysis]] to defensive controls in [[06 - DEFENSIVE SECURITY/Defensive Security - Index]].

---

## Event Log Sources

These Windows Event Log sources provide the primary telemetry for AD attack detection:

| Source | Key Information |
|---|---|
| **Security Event Log (on Domain Controllers)** | Authentication events, privilege use, object access, policy changes |
| **System Event Log** | Service events, system changes |
| **PowerShell Operational Log** | Script block logging, module logging |
| **Sysmon** | Process creation, network connections, registry changes, file activity |
| **ADCS / Certificate Services** | Certificate enrollment and issuance |
| **Windows Defender / EDR** | Endpoint telemetry, process activity, credential access indicators |

> Event log collection must be configured and verified. Logs that are not collected and retained provide no defensive value.

---

## Key Detection Event IDs

| Event ID | Description | Relevant Attack |
|---|---|---|
| 4624 | Successful logon | Lateral movement, pass-the-hash (check logon type and auth package) |
| 4625 | Failed logon | Brute force, password spray |
| 4648 | Logon using explicit credentials | Pass-the-hash, credential reuse |
| 4662 | Object accessed with specific permissions | DCSync (check for replication rights used by non-DC) |
| 4663 | Object access attempt | Sensitive object access |
| 4672 | Special privileges assigned to logon | Privilege escalation; admin logon |
| 4720 | User account created | Persistence via account creation |
| 4728 / 4732 / 4756 | Member added to security-enabled group | Privilege escalation; group modification |
| 4740 | Account locked out | Brute force, password spray |
| 4768 | Kerberos TGT request | AS-REP Roasting (check pre-auth type); logon patterns |
| 4769 | Kerberos service ticket request | Kerberoasting (check encryption type 0x17); lateral movement |
| 4771 | Kerberos pre-auth failed | AS-REP Roasting attempts |
| 4776 | NTLM authentication | Pass-the-hash; NTLM relay; monitor for anomalies |
| 5136 | Directory Service Object modified | GPO modification; ACL change; group membership change |
| 5145 | Network share access check | Lateral movement via shares |
| 7045 | New service installed | Persistence; lateral movement |

> Event IDs and their meanings may vary by Windows version and environment configuration. Validate in your specific environment. Do not deploy detections based solely on this list without testing.

---

## Detection Patterns

### Kerberoasting Detection

**What to look for:**
- Unusual volume of 4769 events (Kerberos service ticket requests) from a single source account
- 4769 events with encryption type 0x17 (RC4-HMAC) for service principal name accounts
- Service ticket requests for accounts not typically used by that source

**False positive risk:** Legitimate applications request many service tickets. Tune on encryption type and anomalous request patterns.

---

### DCSync Detection

**What to look for:**
- 4662 events from non-domain-controller accounts using replication permissions: `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` (DS-Replication-Get-Changes-All)

**False positive risk:** Domain controllers perform replication legitimately. Filter on source being a non-DC computer account.

---

### Pass-the-Hash Detection

**What to look for:**
- 4624 type 3 (network logon) with authentication package NTLM from workstations authenticating to servers
- NTLM authentication from accounts that should use Kerberos
- Logons from a source that has no recent interactive logon on that host

**False positive risk:** NTLM is widely used. Detection requires baselining normal NTLM patterns in the environment.

---

### LDAP Enumeration Detection

**What to look for:**
- High volume of LDAP queries from a single workstation or user account in a short time window
- Queries for sensitive attributes (e.g., `userPassword`, `unixUserPassword`, `ms-DS-ManagedPassword`)
- LDAP queries for all group memberships, delegation settings, or ACLs from non-admin accounts

---

### Suspicious Certificate Enrollment

**What to look for:**
- Certificate enrollment for templates that include client authentication, where the enrolling account is not expected to enroll for that template
- Enrollment events where the Subject Alternative Name contains a different account than the requesting user

---

## Hardening Reference

### Privileged Access Controls

| Control | Action |
|---|---|
| Tiered Administration | Enforce Tier 0/1/2 separation; use Privileged Access Workstations for Tier 0 |
| Protected Users | Add domain admins and sensitive accounts to Protected Users |
| Credential Guard | Enable on domain-joined systems to prevent LSASS credential extraction |
| LSASS Protection | Enable RunAsPPL to protect LSASS process |
| Fine-Grained Password Policies | Apply stronger password requirements to privileged accounts |
| Privileged Access Management | Time-limited privileged access; just-in-time admin |

See: [[Control - Privileged Access Management]]

---

### Authentication Hardening

| Control | Action |
|---|---|
| MFA for Privileged Access | Enforce MFA for all privileged accounts and remote access |
| Kerberos AES Enforcement | Disable RC4 where environment permits; enforce AES encryption |
| NTLM Restriction | Audit and restrict NTLM; block NTLM for domain admin accounts |
| Smart Card / FIDO2 | Enforce phishing-resistant authentication for privileged accounts |
| Legacy Protocol Blocking | Disable NTLMv1; disable LM hashes |

See: [[Control - Multi-Factor Authentication]], [[Control - Secure Remote Access]]

---

### Active Directory Object Hardening

| Area | Action |
|---|---|
| Privileged Group Membership | Audit regularly; remove unnecessary members; document exceptions |
| ACL Review | Periodic review of write permissions on privileged objects; remove GenericWrite, WriteDACL, WriteOwner from non-admin users |
| Delegation Audit | Audit unconstrained delegation; remove from non-DC systems; review RBCD |
| AdminSDHolder | Review AdminSDHolder ACL; remove unnecessary ACEs |
| SYSVOL Audit | Remove credentials from GPP files, scripts, SYSVOL |
| Service Account Management | Use gMSA; audit SPNs; rotate passwords regularly |

---

### ADCS Hardening

| Area | Action |
|---|---|
| Certificate Template Audit | Disable unused templates; restrict enrollment permissions; enable manager approval for sensitive templates |
| SAN Restriction | Disable user-supplied Subject Alternative Name in templates |
| CA Permissions | Restrict who can manage the CA; audit CA ACLs |
| Audit Logging | Enable ADCS audit logging; forward to SIEM |

---

## SIEM Use Cases

Route AD telemetry to [[SIEM - Overview]] and configure correlation rules for:

- Kerberoasting: high-volume RC4 service ticket requests
- DCSync: replication permission use by non-DC accounts
- Privilege escalation: account added to Tier 0 groups
- GPO modification: 5136 events on GPO objects
- Suspicious authentication: NTLM from privileged accounts

See: [[SIEM - Detection Use Cases]], [[Detection Engineering - Lifecycle]]

---

## Related Notes

- [[Active Directory Security - Overview]] — foundational concepts
- [[Active Directory - Attack Path Analysis]] — offensive perspective
- [[GOAD Project - Risk and Control Bridge]] — lab validation
- [[Detection Engineering - Lifecycle]] — how to build and maintain detections
- [[Control - Privileged Access Management]] — PAM control
- [[Control - Multi-Factor Authentication]] — MFA control
