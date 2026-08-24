---
title: Enumeration - Concepts
category: Security Domain
tags:
  - Enumeration
  - OffensiveSecurity
  - AttackSurface
date_created: 2026-08-24
status: MAINTAINED
---

# Enumeration — Concepts

## Purpose

Enumeration is the process of systematically identifying specific services, identities, permissions, and configurations within the authorized scope. Where reconnaissance answers "what is out there?", enumeration answers "what exactly is running, who has access, and what is misconfigured?"

---

## Why Enumeration Matters

Most significant vulnerabilities in enterprise environments are not unpatched CVEs — they are misconfigurations, excessive permissions, and forgotten services discovered through careful enumeration. An attacker who enumerates an environment thoroughly often finds a path to their objective without exploiting a single CVE.

---

## Enumeration Categories

### Service Enumeration

| What | Purpose | Evidence |
|---|---|---|
| Open ports and services | Identify what is listening and reachable | Port scan output with version information |
| Service versions | Identify known vulnerabilities | Banner output; version strings |
| Web application technologies | Framework, server, dependencies | HTTP response headers; `X-Powered-By`; error pages |
| SSL/TLS configuration | Identify weak protocols or certificates | TLS scan output |
| SMB shares and access | Identify accessible file shares | Share enumeration output |
| Database ports and access | Identify exposed database services | Port and authentication enumeration |

---

### Identity and Account Enumeration

| What | Purpose | Evidence |
|---|---|---|
| Domain users | Understand identity landscape; identify high-value targets | LDAP query output; AS-REP enumeration |
| Domain groups and membership | Identify privileged accounts and groups | LDAP query output |
| Service accounts | Identify accounts with SPN registered (Kerberoasting targets) | SPN enumeration output |
| Local accounts | Identify built-in and local administrator accounts | Local account enumeration |
| Email address format | Enable phishing or credential stuffing | Email format inference from discovered accounts |

---

### Permission and ACL Enumeration

| What | Purpose | Evidence |
|---|---|---|
| Active Directory ACLs | Identify write permissions on privileged objects | BloodHound (lab/authorized); manual LDAP queries |
| File share permissions | Identify sensitive data accessible to broad user groups | Share access enumeration output |
| Cloud IAM policies | Identify over-permissioned roles or accounts | Cloud provider IAM enumeration |
| Application role assignments | Identify users with administrative application roles | Application role enumeration |

---

### Configuration and Misconfiguration Enumeration

| What | Purpose | Evidence |
|---|---|---|
| LDAP signing and channel binding | Identify relay attack surface | LDAP configuration output |
| SMB signing | Identify relay attack surface | SMB signing enumeration output |
| Delegation settings | Identify unconstrained delegation; constrained delegation targets | AD attribute enumeration |
| Password policy | Identify spray thresholds and minimum password requirements | Domain password policy query |
| ADCS configuration | Identify certificate templates and enrollment permissions | ADCS enumeration output |
| GPO settings | Identify policy gaps; scripts with credentials | SYSVOL access and GPO review |

---

## Enumeration and Detection

Enumeration generates telemetry that defenders can detect:

| Enumeration Activity | Expected Telemetry |
|---|---|
| LDAP queries for all users | High-volume LDAP bind and search events from workstation |
| SPN enumeration (Kerberoasting prep) | Service ticket requests from workstation accounts at unusual times |
| Share access | Network share access events (5145) from unfamiliar source |
| DNS zone transfer attempt | DNS audit events |
| Port scanning | Network connection logs; IDS/IPS alerts |
| Web application crawling | Web server access logs; WAF events |

> Enumeration activity should be conducted within authorized testing windows and documented with timestamps. Defenders may detect and investigate enumeration as a real incident.

---

## Organizing Enumeration Findings

Enumeration output should be documented into:

| Category | Document |
|---|---|
| Asset inventory delta | Systems and services found that were not in the client's inventory |
| Identity landscape | Users, groups, service accounts, privileged accounts |
| Misconfiguration list | Specific configuration weaknesses requiring follow-up |
| Attack surface for follow-on testing | What to test next, with priority |

---

## Lab Bridge

Active Directory enumeration procedures are validated in the GOAD lab:

| Note | Purpose |
|---|---|
| [[GOAD - Vault Index]] | GOAD navigation |
| [[GOAD - Command Execution Matrix]] | Authorized lab enumeration procedures |

Web application enumeration concepts are applied in [[Web Application Security - Overview]].

---

## Related Notes

- [[Reconnaissance - Concepts]] — prior phase: discovering the attack surface
- [[Vulnerability Assessment - Concepts]] — next phase: confirming and prioritizing weaknesses
- [[Active Directory Security - Overview]] — AD-specific enumeration context
- [[Offensive Security Methodology]] — Phase 6 (Enumeration)
