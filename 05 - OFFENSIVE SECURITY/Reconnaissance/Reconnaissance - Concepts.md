---
title: Reconnaissance - Concepts
category: Security Domain
tags:
  - Reconnaissance
  - AttackSurface
  - OffensiveSecurity
date_created: 2026-08-24
status: MAINTAINED
---

# Reconnaissance — Concepts

## Purpose

Reconnaissance is the phase in which information about the target is gathered before direct interaction with systems begins. The goal is to understand the exposed attack surface and identify where testing effort should be focused.

Reconnaissance is conducted within authorized scope only. The boundary between passive and active reconnaissance matters because active reconnaissance directly interacts with target systems.

---

## Passive vs Active Reconnaissance

| Type | Definition | Contact with Target |
|---|---|---|
| **Passive** | Gathering information from publicly available sources without directly interacting with target systems | None — target cannot observe the activity |
| **Active** | Gathering information through direct interaction with target systems (e.g., port scanning, DNS queries to target servers) | Direct — target systems may log the activity |

> Active reconnaissance should be conducted only within the authorized scope and testing window. Even scanning can cause disruption on sensitive systems.

---

## Business Risk of Exposed Attack Surface

What reconnaissance reveals is not just useful for testers — it reflects what adversaries can discover about the organization from the internet.

| Discovered Information | Business Risk Implication |
|---|---|
| Exposed legacy services on the internet | Immediate exploitation target; no authentication bypass needed |
| Exposed VPN or remote access portals | Credential attack target; phishing target |
| Employee names and email formats | Phishing, social engineering, credential stuffing |
| Technology stack information | Enables targeted exploitation of known vulnerabilities |
| Open cloud storage | Direct data access without authentication |
| Exposed credentials in code repositories | Direct access using real credentials |
| Certificate transparency: subdomains | Attack surface expansion; legacy/shadow systems |
| Job postings detailing internal technology | Targeted attack planning |

> If a tester can find it in 30 minutes using public sources, a motivated adversary can find it before you know they are looking.

---

## Passive Reconnaissance Sources

| Source | What It Reveals |
|---|---|
| **DNS records** | Subdomains, mail servers, name servers, IP ranges |
| **Certificate Transparency (e.g., crt.sh)** | All certificates issued for a domain; reveals subdomains |
| **WHOIS / RDAP** | Domain registration details, registrar, registration dates |
| **Shodan / Censys** | Internet-exposed services, versions, banners, open ports |
| **Google/Bing** | Exposed files, error messages, login portals, directory listings |
| **LinkedIn / Social Media** | Employee names, roles, technologies mentioned, organizational structure |
| **Job postings** | Internal technology stack, processes, tools, team structure |
| **GitHub / GitLab / Code Repositories** | Hardcoded credentials, internal hostnames, configuration files, API keys |
| **Wayback Machine** | Historical versions of web content; removed pages with sensitive information |
| **Email headers** | Mail server infrastructure; SPF/DKIM configuration |

---

## Active Reconnaissance (Authorized)

When active reconnaissance is authorized:

| Activity | Purpose | Risk |
|---|---|---|
| DNS resolution and zone transfer attempt | Confirm DNS records; identify misconfigurations | Zone transfer requests are logged |
| Network scanning (ping sweep, port scan) | Identify live hosts and open ports within scope | Scanning generates logs; aggressive scanning can disrupt services |
| Service banner grabbing | Identify software and versions | Interacts with target services |
| Web crawling | Map application structure | Web server logs record requests |

> Always confirm active reconnaissance is within the authorized scope and testing window. Coordinate with operations if scanning production IP ranges.

---

## Attack Surface Analysis

Reconnaissance output should be organized into an attack surface picture:

| Category | Examples |
|---|---|
| **External Network** | IP ranges, ASN, cloud provider IP space |
| **External Services** | HTTPS, VPN, RDP, SMTP, SSH, FTP — exposed to internet |
| **Web Applications** | Public-facing applications; login portals; APIs |
| **Subdomains** | All subdomains discovered; shadow/legacy systems |
| **Identity Exposure** | Employee emails; credential exposure in public repositories |
| **Technology Stack** | Server software; frameworks; libraries |
| **Cloud Footprint** | Cloud provider; public storage buckets; exposed cloud APIs |
| **Third-Party Risk** | Partner portals; CDN; supply chain exposure |

---

## Evidence

Reconnaissance evidence should include:

- Source of each finding (specific URL, tool, query)
- Date of discovery
- Screenshot or output where the information was found
- Assessment of business relevance

---

## Lab Bridge

Asset discovery and reconnaissance concepts are applied in the GOAD laboratory within the authorized network:

- [[GOAD - Vault Index]] — GOAD environment navigation
- [[GOAD Project - Risk and Control Bridge]] — how GOAD reconnaissance maps to business risk

---

## Related Notes

- [[Enumeration - Concepts]] — next step: enumerate discovered assets in detail
- [[Offensive Security Methodology]] — Phase 4 (Asset Discovery), Phase 5 (Reconnaissance)
- [[Offensive Security Safety and Authorization]] — authorization requirements before active reconnaissance
