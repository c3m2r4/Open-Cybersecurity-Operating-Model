---
title: GOAD - Services - MSSQL
category: Services
environment: GOAD
status: TODO
tags:
  - GOAD
  - MSSQL
  - Services
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 7; Part 12 (MSSQL trust links)"
version_compatibility: GOAD v2 / LAB-MODIFIED / VERIFY
---

# GOAD — MSSQL Services

> [!WARNING]
> **TODO — Procedures not yet written.** This note defines coverage requirements only.
> Do not execute techniques from external references without verifying your deployment.

> [!CAUTION]
> Mayfly Part 7 states MSSQL configuration requires lab updates (`ansible-playbook servers.yml`).
> **Verify MSSQL impersonation/link configuration exists in your deployment before attempting any procedure.**

> [!IMPORTANT]
> External reference: [Mayfly GOAD Part 7 — MSSQL](https://mayfly277.github.io/posts/GOADv2-pwning-part7/)
> Use for **topic coverage only** — not as a command source. All IPs, credentials, and hostnames must be replaced with verified lab values.

---

## Scope

Document MSSQL attack surface on GOAD member servers (documented candidates: `<LAB_HOST>` at `192.168.56.22`, `192.168.56.23` — VERIFY with port scan).

---

## Coverage Checklist (To Document)

| Area                                                                 | Status | Matrix Phase (planned)                             |
| -------------------------------------------------------------------- | ------ | -------------------------------------------------- |
| MSSQL discovery (port 1433, SPN via GetUserSPNs)                     | TODO   | Phase 1 / Phase 2                                  |
| MSSQL enumeration (CME, impacket mssqlclient)                        | TODO   | Phase 2                                            |
| Authentication (domain creds, Windows auth)                          | TODO   | Phase 2 / Phase 3                                  |
| Login vs user impersonation (`EXECUTE AS LOGIN` / `EXECUTE AS USER`) | TODO   | Phase 6 / Phase 8                                  |
| xp_cmdshell / trustworthy database property                          | TODO   | Phase 8                                            |
| Linked servers / trusted links (`sp_linkedservers`)                  | TODO   | Phase 7 / Phase 8                                  |
| Cross-domain / cross-forest MSSQL links                              | TODO   | Phase 8 — see [[GOAD - Active Directory - Trusts]] |
| MSSQL coerce (xp_dirtree NTLM)                                       | TODO   | See [[GOAD - Enumeration - Poisoning and Relay]]   |
| Evidence collection                                                  | TODO   | Phase 9                                            |
| Cleanup (xp_cmdshell artifacts, uploaded files)                      | TODO   | Phase 10                                           |

---

## Version / Lab Compatibility

| Classification | Applies To |
|---|---|
| GOAD v2 | Base MSSQL on CASTELBLACK / BRAAVOS topology |
| LAB-MODIFIED | Impersonation privileges (arya.stark/msdb, brandon.stark/jon.snow) per Mayfly — **VERIFY** |
| VERSION-DEPENDENT | impacket mssqlclient extensions — verify tool version before use |
| UNKNOWN | Whether linked server BRAAVOS mapping exists in current deployment |

---

## Execution Context (When Written)

| Activity | Expected Context |
|---|---|
| Port scan / CME / GetUserSPNs | `[KALI]` — Bash |
| mssqlclient enumeration | `[KALI]` — Bash |
| xp_cmdshell output | Record via Mythic callback or remote shell — document per access vector |

---

## Prerequisites (When Written)

- [ ] Phase 0 complete — [[GOAD - IP Hostname Matrix]] verified
- [ ] Port 1433 confirmed open on target `<LAB_HOST>` — VERIFY
- [ ] Valid domain credentials or callback context — VERIFY
- [ ] Lab MSSQL configuration confirmed (not assumed from external write-up)

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Enumeration - Initial Recon]]
- [[GOAD - Active Directory - Trusts]]
- [[GOAD - Enumeration - Poisoning and Relay]]
- [[Operation Cypher-Knife - Findings]]
