---
title: GOAD - Active Directory - ADCS
category: Active Directory
environment: GOAD
status: TODO
tags:
  - GOAD
  - ADCS
  - PKI
  - Certificates
  - ESC
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 6; Part 14 (ADCS 5/7/9/10/11/13/14/15)"
version_compatibility: GOAD v2 / LAB-MODIFIED / VERSION-DEPENDENT
---

# GOAD — Active Directory Certificate Services (ADCS)

> [!WARNING]
> **TODO — Procedures not yet written.** ADCS is a major coverage gap in the maintained vault.
> ESC availability varies by GOAD version, provisioning, and lab modifications.

> [!CAUTION]
> Mayfly Part 14 explicitly states GOAD was **modified** to support additional ESC scenarios.
> Provisioning requires: `ad-data.yml`, `ad-relations.yml`, `adcs.yml`, `vulnerabilities.yml` (GOAD 3.x CLI) or equivalent ansible playbooks.
> **Do not add ESC techniques to the normal attack path until confirmed present in your deployment.**

> [!IMPORTANT]
> External references:
> - [Mayfly Part 6 — ADCS (ESC1–4, ESC6, ESC8, Certifried, Shadow Credentials)](https://mayfly277.github.io/posts/GOADv2-pwning-part6/)
> - [Mayfly Part 14 — ADCS ESC5/7/9/10/11/13/14/15](https://mayfly277.github.io/posts/ADCS-part14/)

---

## ADCS Discovery & Enumeration (To Document)

| Area | Status |
|---|---|
| ADCS presence detection (LDAP, CME adcs module, certipy find) | TODO |
| CA enumeration (`<LAB_CA>` on `<LAB_HOST>`) | TODO |
| Certificate template enumeration | TODO |
| Vulnerable template identification (certipy `-vulnerable`) | TODO |
| BloodHound PKI / certipy BloodHound export | TODO |
| Web enrollment check (HTTP/HTTPS on `<LAB_HOST>`) | TODO |
| PKINIT / certificate authentication concepts | TODO — conceptual only until verified |
| ADCS detection indicators | TODO |
| ADCS remediation notes (lab rollback) | TODO |

---

## ESC Technique Coverage Matrix

> **Do not assume any ESC condition exists until verified in your lab.**
> Check one box per technique after live enumeration — not from this document.

| ESC | Description (conceptual) | Mayfly Ref | GOAD Lab Status |
|---|---|---|---|
| ESC1 | Enrollee supplies subject + client auth template | Part 6 | [ ] Confirmed present [ ] Not confirmed [ ] Requires lab modification [ ] Not applicable |
| ESC2 | Any Purpose EKU on template | Part 6 | [ ] Confirmed present [ ] Not confirmed [ ] Requires lab modification [ ] Not applicable |
| ESC3 | Certificate Request Agent / on-behalf-of | Part 6 | [ ] Confirmed present [ ] Not confirmed [ ] Requires lab modification [ ] Not applicable |
| ESC4 | Vulnerable template ACL (GenericWrite → modify template) | Part 6 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC5 | CA certificate / golden certificate | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC6 | EDITF_ATTRIBUTESUBJECTALTNAME2 on CA | Part 6 | [ ] Confirmed present [ ] Not confirmed [ ] Requires lab modification [ ] Not applicable |
| ESC7 | Vulnerable CA ACL (manage CA) | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC8 | NTLM relay to HTTP web enrollment | Part 6 | [ ] Confirmed present [ ] Not confirmed [ ] Requires lab modification [ ] Not applicable |
| ESC9 | No security extension + client auth | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC10 | Weak certificate mapping (ESC6 variant) | Part 14 | [ ] Confirmed present [ ] VERSION-DEPENDENT [ ] Not confirmed [ ] Not applicable |
| ESC11 | Relay to ICPR (RPC) | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC13 | Issuance policy / OID group link | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC14 | VA replication / certifried variant | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |
| ESC15 | Arbitrary application policy + user SAN | Part 14 | [ ] Confirmed present [ ] Requires lab modification [ ] Not confirmed [ ] Not applicable |

### Related Non-ESC Techniques (Mayfly Part 6)

| Technique | Classification | GOAD Lab Status |
|---|---|---|
| Certifried (CVE-2022-26923) | VERSION-DEPENDENT | [ ] Not confirmed [ ] Not applicable |
| Shadow Credentials (GenericWrite + ADCS) | Requires ADCS + Write ACL | [ ] Not confirmed — see [[GOAD - Active Directory - ACL and ACE Abuse]] |

---

## Lab Modification Warning (Part 14)

> [!WARNING]
> This technique requires a lab modification described by the external reference.
> Mayfly Part 14 provisioning (GOAD 3.x example):
> ```text
> provision ad-data.yml
> provision ad-relations.yml
> provision adcs.yml
> provision vulnerabilities.yml
> ```
> Verify that the modification exists in the current deployment before attempting ESC5/7/9/10/11/13/14/15 procedures.

---

## Version / Lab Compatibility Summary

| Classification | Items |
|---|---|
| CURRENT GOAD | ADCS installed on essos forest (typical GOAD v2) — **VERIFY** |
| LAB-MODIFIED | ESC4 template ACL, ESC5 CA admin, Part 14 ESC scenarios |
| VERSION-DEPENDENT | Certifried, ESC10 weak mapping, PetitPotam unauthenticated (patched on current AD) |
| MAYFLY GOAD | Bot/simulated LLMNR users, specific template names — do not copy |

---

## Execution Context (When Written)

| Activity | Context |
|---|---|
| certipy / CME adcs from Kali | `[KALI]` |
| PKINIT / gettgtpkinit | `[KALI]` |
| Web enrollment relay | `[KALI]` + network positioning — see [[GOAD - Enumeration - Poisoning and Relay]] |
| Template modification | `[KALI]` — **modifies lab** |
| Certificate auth to LDAP shell | `[KALI]` |

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Active Directory - ACL and ACE Abuse]]
- [[GOAD - Enumeration - Poisoning and Relay]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
