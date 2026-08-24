---
title: Operation Cypher-Knife - Scope
category: Engagement
environment: GOAD
tags:
  - OperationCypherKnife
  - Scope
  - Engagement
date_created: 2026-08-23
status: ACTIVE
---

# Operation Cypher-Knife — Scope

> [!WARNING]
> This operation is conducted within the **authorized GOAD lab environment only**.
> Network: 192.168.56.0/24 (VirtualBox host-only — not internet-routable).

---

## Engagement Overview

| Property | Value |
|---|---|
| Operation Name | Operation Cypher-Knife |
| Environment | GOAD (Game of Active Directory) |
| Network | 192.168.56.0/24 |
| Purpose | Cybersecurity training — authorized lab |
| Authorization | Lab owner (self-authorized) |

---

## In-Scope Systems

| Host | IP | Domain | Status |
|---|---|---|---|
| King's Landing | 192.168.56.10 | sevenkingdoms.local | VERIFIED |
| Winterfell | 192.168.56.11 | north.sevenkingdoms.local | VERIFIED |
| Meereen | 192.168.56.12 | essos.local | VERIFIED |
| Castle Black | 192.168.56.22 | north.sevenkingdoms.local | VERIFIED |
| Braavos | 192.168.56.23 | essos.local | VERIFIED |

> [!NOTE]
> All IPs verified from live GOAD deployment nmap scans. See [[GOAD - IP Hostname Matrix]].

---

## Out of Scope

- Any system outside 192.168.56.0/24
- Any public IP address
- Any real organization's infrastructure
- Any production systems

---

## Objectives

1. Gain initial access to at least one GOAD host
2. Escalate privileges to SYSTEM on the initial host
3. Dump credentials from the initial host
4. Achieve lateral movement to additional hosts
5. Achieve Domain Admin on at least one domain
6. Perform DCSync on at least one domain
7. Document attack paths and findings in BloodHound

---

## Related Notes

- [[Operation Cypher-Knife - Findings]]
- [[Operation Cypher-Knife - Lessons Learned]]
- [[GOAD - Final Validation Checklist]]
- [[GOAD - Error Audit and Contradiction Report]]
