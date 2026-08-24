---
title: GOAD - Enumeration - Poisoning and Relay
category: Enumeration
environment: GOAD
status: TODO
tags:
  - GOAD
  - Poisoning
  - Relay
  - LLMNR
  - NBT-NS
  - Responder
  - TODO
date_created: 2026-08-23
external_reference: "Mayfly GOAD Part 4; Part 13 (coerce files)"
version_compatibility: GOAD v2 / LAB-MODIFIED
---

# GOAD — Poisoning and Relay

> [!WARNING]
> **TODO — Procedures not yet written.** Poisoning/relay affects network-wide authentication traffic.
> Only document tools and techniques explicitly approved for this lab.

> [!CAUTION]
> ERROR 06 in [[GOAD - Error Audit and Contradiction Report]] documents hardcoded `eth0` in old notes.
> Interface must be verified with `ip addr` on Kali — do not copy Mayfly's `vboxnet0` / `tun0` assumptions.

> [!IMPORTANT]
> External references:
> - [Mayfly Part 4 — Poison and Relay](https://mayfly277.github.io/posts/GOADv2-pwning-part4/)
> - [Mayfly Part 13 — Coerce files / WebDAV](https://mayfly277.github.io/posts/GOADv2-pwning-part13/)
> Old vault notes (GOAD Assault Playbook, Cypher-Knife runbook) contain Responder/relay commands marked **DO NOT USE** without verification.

---

## Conceptual Coverage (To Document)

| Area | Status | Notes |
|---|---|---|
| Name-resolution poisoning (LLMNR, NBT-NS, mDNS) | TODO | Mayfly Part 4 — GOAD has simulated bots (**LAB-MODIFIED**) |
| IPv6 DHCP/DNS poisoning (mitm6) | TODO | Mayfly Part 4 |
| Relay prerequisites (SMB signing off, LDAP signing) | Partial | CME signing noted in [[GOAD - Enumeration - Initial Recon]] Step 4 |
| Network positioning (same broadcast domain) | Partial | Phase 0 connectivity only |
| Authentication coercion (PetitPotam, PrinterBug, DFSCoerce, coercer) | TODO | Mayfly Parts 4, 6, 13 |
| Relay targets (SMB, LDAP/LDAPS, HTTP/ADCS) | TODO | Links to [[GOAD - Active Directory - ADCS]] |
| Defensive controls (SMB signing, EPA, LDAP signing) | TODO | Document as validation steps |
| Evidence (NetNTLM captures, relay logs) | TODO | Phase 9 |
| Cleanup (Responder.db, dropped .lnk/.scf/.url files) | TODO | Phase 10 |

---

## Maintained Vault — What Exists Today

| Item | Location | Status |
|---|---|---|
| SMB signing enumeration | [[GOAD - Enumeration - Initial Recon]] | Partial |
| Responder hardcoded interface warning | [[GOAD - Error Audit and Contradiction Report]] ERROR 06 | Documented |
| Responder/relay commands | Old runbooks only | **Not in maintained path** |

---

## Tool Policy

> [!NOTE]
> The maintained vault does **not** currently authorize specific poisoning/relay tools.
> When writing procedures, explicitly list approved tools per engagement scope.
> Do not add Responder or ntlmrelayx commands to the matrix until this note is completed and validated.

Potential tools (reference only — not approved until documented):

- Responder, mitm6, ntlmrelayx, coercer, petitpotam
- CME modules: slinky, scuffy, drop-sc, webdav

---

## Version / Lab Compatibility

| Item | Classification |
|---|---|
| LLMNR/NBT-NS bots (robb.stark, eddard.stark) | **LAB-MODIFIED** — Mayfly Part 4 |
| mitm6 WPAD → LDAPS RBCD on braavos | **LAB-MODIFIED** — DNS config may differ |
| Coerce file drops (.lnk, .scf, .url) | GOAD v2 — Part 13 |
| WebClient on server (braavos) | **LAB-MODIFIED** — custom GOAD vulnerability |

---

## Related Notes

- [[GOAD - Command Execution Matrix]]
- [[GOAD - Enumeration - Initial Recon]]
- [[GOAD - Active Directory - ADCS]]
- [[GOAD - Active Directory - Delegation]]
- [[GOAD - Error Audit and Contradiction Report]]
