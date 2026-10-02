---
title: "API Security in Banking: Protecting the Digital Financial Ecosystem"
subtitle: Board-Level Presentation — Enterprise Cybersecurity & Technology Risk
author: Information Security / IT Risk Management
date: 2026-Q4
classification: CONFIDENTIAL — MANAGEMENT USE ONLY
tags:
  - API Security
  - Board Presentation
  - Technology Risk
  - Cybersecurity
  - Application Security
  - Open Banking
  - OWASP
  - Zero Trust
  - Governance
audience:
  - Board of Directors
  - Senior Management
  - Chief Risk Officer
  - Chief Information Security Officer
  - Chief Technology Officer
  - Internal Audit
  - Compliance
  - IT Risk Management
  - SOC / Security Operations
version: "1.0"
status: DRAFT — FOR REVIEW
related:
  - "[[09 - APPLICATION & DEVSECOPS/Secure SDLC]]"
  - "[[09 - APPLICATION & DEVSECOPS/Threat Modeling]]"
  - "[[06 - DEFENSIVE SECURITY/SIEM/SIEM - Detection Use Cases]]"
  - "[[02 - IT RISK MANAGEMENT]]"
  - "[[08 - SECURITY ARCHITECTURE]]"
---

> [!IMPORTANT]
> **Classification:** This document is CONFIDENTIAL and intended for senior management, IT Risk, Information Security, Internal Audit, and Compliance review. Attack scenarios described are hypothetical risk illustrations based on publicly documented industry patterns. No specific bank vulnerabilities are described or implied.

---

# API Security in Banking: Protecting the Digital Financial Ecosystem

**Enterprise Risk Briefing | Board & Senior Management | 2026-Q4**

---

## Table of Contents

1. [[#Slide 01 — Executive Summary]]
2. [[#Slide 02 — What Is an API? The Business Perspective]]
3. [[#Slide 03 — The Bank's API Ecosystem]]
4. [[#Slide 04 — Why API Security Is a Banking Priority]]
5. [[#Slide 05 — The Expanding API Attack Surface]]
6. [[#Slide 06 — API Threat Landscape — OWASP API Security Top 10]]
7. [[#Slide 07 — Attack Scenarios — What Could Go Wrong?]]
8. [[#Slide 08 — The API Attack Chain]]
9. [[#Slide 09 — API Security vs Traditional Web Security]]
10. [[#Slide 10 — The API Estate Problem]]
11. [[#Slide 11 — Enterprise API Inventory Model]]
12. [[#Slide 12 — Secure API Lifecycle]]
13. [[#Slide 13 — Enterprise API Security Architecture]]
14. [[#Slide 14 — Authentication vs Authorization]]
15. [[#Slide 15 — Zero Trust Principles for APIs]]
16. [[#Slide 16 — API Monitoring and Detection]]
17. [[#Slide 17 — API Security and Fraud]]
18. [[#Slide 18 — Third-Party and Partner API Risk]]
19. [[#Slide 19 — API Security Governance Framework]]
20. [[#Slide 20 — API Risk Classification Model]]
21. [[#Slide 21 — API Security Maturity Model]]
22. [[#Slide 22 — Banking Incident Case Studies]]
23. [[#Slide 23 — Regulatory and Framework Mapping]]
24. [[#Slide 24 — Executive KPIs and KRIs]]
25. [[#Slide 25 — Recommended Roadmap]]
26. [[#Slide 26 — Top 10 Actions for the Bank]]
27. [[#Slide 27 — Final Executive Message]]
28. [[#Annex A — IT Risk API Security Assessment Checklist]]
29. [[#Annex B — API Governance RACI]]
30. [[#Annex C — References]]

---

## Slide 01 — Executive Summary

### Executive Takeaway
> *APIs are the invisible backbone of modern banking. Inadequately secured APIs represent a direct pathway to customer data, financial transactions, and core business systems.*

### Key Points

- The bank's digital services — mobile banking, open banking, payments, partner integrations — are **delivered through APIs**.
- APIs are **business transaction interfaces**: they process payments, authenticate customers, expose account data, and connect trusted systems.
- Globally, **API-related security incidents have increased significantly** across financial services, with attackers specifically targeting API vulnerabilities to conduct fraud, data theft, and account takeover.
- Unlike traditional cyberattacks, API abuse often uses **legitimate credentials and standard protocol**, making it harder to detect.
- This briefing presents the enterprise risk picture, recommended controls, a governance framework, and a practical roadmap.
- **Action Required:** Leadership endorsement of an enterprise API security programme covering discovery, assessment, protection, detection, governance, and continuous improvement.

### Business Risk Statement
> APIs are no longer solely a developer concern. They are an **enterprise cybersecurity, technology risk, operational resilience, third-party risk, fraud, and governance responsibility.**

---

## Slide 02 — What Is an API? The Business Perspective

### Executive Takeaway
> *An API is a digital contract — it defines what business functionality systems can request from each other, what data can be exchanged, and under what conditions.*

### Simple Definition

**API** = *Application Programming Interface*

A structured set of rules that allows software applications to **communicate, exchange data, and invoke each other's functions** in a standardised, machine-readable way.

### Business Analogy

> Think of an API as a **bank teller window** — but operating at machine speed, 24 hours a day, at unlimited scale. The window defines what requests are accepted, what information is returned, and what transactions can be performed. If the window's rules are poorly designed or enforced, unintended transactions become possible.

### What APIs Enable in Banking

| Business Function | How APIs Enable It |
|---|---|
| Mobile banking | App communicates with core banking via secure APIs |
| Payment processing | Payment instruction transmitted via API to payment platform |
| Card authorisation | API call to authorisation engine in milliseconds |
| Open Banking | Regulated API access for TPPs and fintechs |
| Fraud detection | Real-time API calls to fraud scoring engine |
| Identity verification | API integration with identity provider / KYC service |
| Partner integrations | API connections to correspondent banks, insurers, merchants |
| Internal microservices | Internal APIs connecting backend systems |

### The Critical Insight

> **"Every API can become a pathway to business functionality, sensitive data, or financial transactions."**
> — OWASP API Security Project

### Speaker Notes
- APIs are not merely "technical plumbing." They represent the bank's business logic exposed in a structured, programmatic form.
- A poorly secured API is not a theoretical technology risk — it is a direct business risk: an attacker who can manipulate an API may be able to query accounts, move money, or harvest customer data.
- Boards should understand that when the bank deploys a mobile banking app, the visible frontend (the app) is secured by the API backend — and it is the backend API that processes the actual transaction.

---

## Slide 03 — The Bank's API Ecosystem

### Executive Takeaway
> *The bank operates a complex, interconnected API ecosystem. Every connection represents a trust relationship that must be explicitly governed.*

### The Banking API Landscape

```
                    ┌─────────────────────────────────────┐
                    │         EXTERNAL CONSUMERS          │
                    │  Mobile Apps │ Web Portal │ Partners │
                    │  Fintechs   │ Merchants  │ Aggregators│
                    └──────────────────┬──────────────────┘
                                       │
                    ┌──────────────────▼──────────────────┐
                    │          API GATEWAY LAYER          │
                    │  Authentication │ Rate Limiting │ WAF │
                    └──────────────────┬──────────────────┘
                                       │
          ┌────────────────────────────┼───────────────────────────┐
          │                           │                           │
    ┌─────▼──────┐           ┌────────▼───────┐         ┌────────▼────────┐
    │  PAYMENTS  │           │  CORE BANKING  │         │   IDENTITY/IAM  │
    │  PLATFORM  │           │    SYSTEM      │         │    SERVICES     │
    └─────┬──────┘           └───────┬────────┘         └────────┬────────┘
          │                          │                           │
    ┌─────▼──────┐           ┌───────▼────────┐         ┌───────▼─────────┐
    │  CARD &    │           │  FRAUD &       │         │   DATA &        │
    │  ATM/POS   │           │  AML SYSTEMS   │         │   ANALYTICS     │
    └────────────┘           └────────────────┘         └─────────────────┘
```

### Connected Systems

**Internal Systems**
- Core banking platform
- Payment & settlement systems
- Card management & authorisation
- ATM/POS switching
- Fraud detection & AML
- CRM & customer data
- Identity & authentication services
- Data platforms & analytics

**External / Third-Party**
- Fintech partners & aggregators
- Payment network APIs (Visa/Mastercard/SWIFT)
- Open Banking TPPs (Third-Party Providers)
- Cloud service providers
- KYC/identity verification vendors
- Correspondent banking institutions
- Government & regulatory reporting systems
- Digital wallet providers

### Key Risk Observation
Every integration = a trust relationship. Every trust relationship must be formally assessed, authorised, monitored, and maintained.

---

## Slide 04 — Why API Security Is a Banking Priority

### Executive Takeaway
> *APIs directly expose business functionality and financial transactions. A security weakness in an API is a weakness in the bank's core business operations.*

### Why APIs Are High-Value Targets

| Factor | Why It Matters |
|---|---|
| **Direct business function access** | APIs process real transactions, not just data |
| **Machine-speed scale** | Thousands of API calls per second — attackers can automate abuse at scale |
| **Authentication / authorisation centrality** | APIs control who can do what — weaknesses here are catastrophic |
| **Multi-party trust chains** | APIs connect the bank to partners, fintechs, and third parties |
| **Bypass of traditional controls** | API traffic may bypass WAF rules designed for web pages |
| **Mobile & partner consumption** | APIs consumed by mobile apps and partners increase the attack surface |
| **Real-time payments** | Real-time payment APIs offer near-instant, potentially irreversible financial outcomes |
| **Open Banking regulation** | Mandated API exposure increases the regulated attack surface |

### The Critical Distinction

| Compromising a Website | Abusing an API |
|---|---|
| Typically requires exploiting a vulnerability | May use **legitimate credentials and valid protocol** |
| Usually detectable through web WAF | Standard API traffic — harder to distinguish from legitimate use |
| Impacts the web layer | **Directly impacts business logic and data** |
| Often requires custom malware | **No malware required** — just API knowledge |

> **An attacker does not necessarily need to "hack the bank" in the traditional sense. They may find a legitimate API and manipulate how it is used.**

---

## Slide 05 — The Expanding API Attack Surface

### Executive Takeaway
> *Digital transformation has dramatically increased the number of APIs, the diversity of consumers, and the complexity of trust relationships — all of which expand the bank's attack surface.*

### Drivers of API Surface Expansion

```
Digital Banking          → New customer-facing APIs
Mobile-First Strategy    → APIs consumed by millions of devices
Open Banking             → Regulated external API exposure
Fintech Partnerships     → External parties accessing internal systems via API
Cloud Adoption           → Cloud-native APIs and cloud provider APIs
Microservices            → Hundreds of internal service-to-service APIs
Real-Time Payments       → High-value, time-sensitive API transactions
Banking-as-a-Service     → Bank's capabilities exposed to third parties
AI-Enabled Applications  → API-driven AI/ML inference services
Embedded Finance         → Bank APIs embedded in non-bank digital products
```

### The Evolving Security Challenge

**Previously:** *"Are our applications secure?"*

**Now:** *"Do we know every API exposed by the bank, who can access it, what it can do, what data it exposes, and whether that access is appropriate?"*

### Risk Implication
The challenge is not just securing individual APIs. It is **governing the entire API estate** as a strategic risk domain.

---

## Slide 06 — API Threat Landscape — OWASP API Security Top 10

### Executive Takeaway
> *The OWASP API Security Top 10 (2023) is the globally recognised authoritative framework for API-specific risks. Banking environments are exposed to every category.*

> [!NOTE]
> **Source:** OWASP API Security Top 10 — 2023 Edition. https://owasp.org/API-Security/editions/2023/en/0x11-t10/

### OWASP API Security Top 10 — Banking Risk Mapping

| # | Risk | Banking Example | Potential Impact | Primary Control |
|---|---|---|---|---|
| **API1** | Broken Object-Level Authorization (BOLA) | Customer modifies account ID to view another customer's account | Unauthorised data access, privacy breach, regulatory exposure | Object-level authorisation checks per authenticated user |
| **API2** | Broken Authentication | Weak token validation allows session hijacking; API key embedded in mobile app | Account takeover, unauthorised transactions | Strong token management, short-lived tokens, MFA |
| **API3** | Broken Object Property Level Authorization | API exposes hidden fields (credit limit, internal flags) in returned objects | Internal data leakage, data manipulation | Field-level authorisation; response filtering |
| **API4** | Unrestricted Resource Consumption | Automated API calls exhaust infrastructure, causing denial of service | Operational disruption, infrastructure cost, service unavailability | Rate limiting, quotas, throttling |
| **API5** | Broken Function-Level Authorization | Customer discovers admin API endpoint and invokes account management functions | Privilege escalation, unauthorised account operations | Role-based function access controls, endpoint segregation |
| **API6** | Unrestricted Access to Sensitive Business Flows | Attacker automates loan application API to extract approved credit decisions | Business logic abuse, fraud | Business flow controls, velocity checks, behavioural monitoring |
| **API7** | Server-Side Request Forgery (SSRF) | API processes external URLs — attacker causes server to access internal systems | Access to internal infrastructure, credential theft | Input validation, network egress controls, URL allowlisting |
| **API8** | Security Misconfiguration | API returns stack traces, debug info, or unnecessary HTTP methods are enabled | Information disclosure enabling targeted attacks | Secure configuration standards; security testing |
| **API9** | Improper Inventory Management | Deprecated API version (v1) remains accessible after v2 deployment | Exploitation of unpatched legacy endpoints | API inventory management, formal deprecation process |
| **API10** | Unsafe Consumption of Third-Party APIs | Bank trusts third-party API response without validation — attacker compromises third party | Supply chain attack, data integrity breach | Third-party API validation, security requirements for vendors |

### Speaker Notes
- BOLA (API1) is the most prevalent API vulnerability — consistently the highest-frequency finding in API penetration testing.
- API9 (Improper Inventory) is particularly relevant to banking environments with long-lived legacy systems and multiple API versions.
- Source: OWASP API Security Project (2023): https://owasp.org/API-Security/

---

## Slide 07 — Attack Scenarios — What Could Go Wrong?

### Executive Takeaway
> *API vulnerabilities translate directly into fraud, data breaches, account takeover, and regulatory exposure. The following are realistic risk scenarios based on documented industry patterns.*

> [!NOTE]
> All scenarios below are hypothetical risk illustrations based on documented API vulnerability patterns from OWASP, NIST, and publicly reported industry incidents. They do not describe actual incidents at this bank.

---

### Scenario 1 — Broken Object-Level Authorization (BOLA)

**What happens:**
A legitimate authenticated customer makes an API request:
`GET /api/v1/accounts/12345678/transactions`
The attacker changes the account number:
`GET /api/v1/accounts/12345679/transactions`
The API returns the other customer's transaction history because it validates **authentication** but not **whether the authenticated user owns account 12345679**.

**Why authentication alone doesn't prevent it:**
Authentication confirms identity. It does not automatically confirm ownership or access rights to a specific object.

**Impact:**
- Exposure of customer transaction history, balances, and personal data
- Privacy and data protection regulatory breach
- Customer trust and reputational damage
- Potential for targeted fraud using harvested account data

**Key Control:**
Every API request for a specific object must verify that the **authenticated user is authorised to access that specific object** — not just that they are authenticated.

---

### Scenario 2 — Broken Function-Level Authorization

**What happens:**
A customer discovers through API enumeration that an administrative endpoint exists:
`POST /api/v1/admin/accounts/freeze`
The customer invokes the endpoint with a valid token and successfully freezes another customer's account.

**Impact:**
- Unauthorised account operations
- Privilege escalation
- Operational disruption
- Potential for targeted denial-of-service against specific customers

**Key Control:**
API endpoints must enforce role-based access controls. Administrative functions must never be accessible to standard user credentials, even if the endpoint URL is known.

---

### Scenario 3 — Excessive Data Exposure

**What happens:**
A mobile banking API returns a full customer object including:
- Account balances across all accounts
- Internal credit flags and scoring
- Customer internal reference IDs
- Linked account details
- Historical PAN fragments

The mobile app displays only the account balance — but the full data object is visible in the API response to anyone intercepting or inspecting it.

**Impact:**
- Large-scale customer data harvesting
- Internal system information disclosure
- Regulatory breach (data minimisation obligations)
- Facilitates targeted fraud and account takeover

**Key Control:**
APIs must return **only the minimum data necessary** for the specific business function. Response filtering must be applied server-side, not client-side.

---

### Scenario 4 — Business Logic Abuse

**What happens:**
A real-time payment API correctly authenticates and authorises individual transactions. However, it does not limit the **frequency** or **cumulative value** of transfers to a new beneficiary within a session.

An attacker with a compromised credential makes 200 small transfers to an external account within minutes, staying below individual transaction monitoring thresholds.

**Impact:**
- Financial loss
- Fraud
- Customer harm
- AML/CTF compliance exposure

**Key Control:**
Business logic controls — velocity checks, cumulative value limits, new-beneficiary cooling periods, and behavioural monitoring — must complement authentication controls.

---

### Scenario 5 — Shadow / Undocumented APIs

**What happens:**
A development team created an API endpoint during testing: `/api/test/accounts/export`. The endpoint exports all accounts in a given range in CSV format and was never disabled before production deployment.

**Impact:**
- Mass data exfiltration
- Regulatory breach
- Reputational damage
- The endpoint is not monitored because it is not in the official API inventory

**Key Control:**
All API endpoints — including test, development, legacy, and deprecated — must be formally inventoried. All non-production endpoints must be **confirmed disabled** before go-live.

---

### Scenario 6 — Third-Party API Compromise

**What happens:**
The bank integrates with a third-party KYC verification vendor via API. The vendor suffers a breach and attackers intercept or manipulate KYC verification responses. The bank's onboarding API trusts the vendor response without independent validation.

**Impact:**
- Fraudulent customer onboarding
- AML/KYC regulatory exposure
- Customer data compromise
- Indirect pathway into bank data flows

**Key Control:**
Third-party API responses must be validated for integrity and consistency. Vendor security requirements, contractual obligations, and security assessments must be maintained.

---

### Scenario 7 — Credential / API Key Leakage

**What happens:**
A developer commits an API key to a public GitHub repository as part of a CI/CD configuration file. An automated secret-scanning tool used by attackers finds the key within minutes.

**Impact:**
- Unauthorised access to internal APIs
- Potential access to customer data or administrative functions
- Secrets exposure is often permanent if not detected and rotated immediately

**Key Control:**
Secrets must never be embedded in code. Secrets management platforms must be mandated. Automated pre-commit scanning must detect secrets before they reach repositories.

---

## Slide 08 — The API Attack Chain

### Executive Takeaway
> *Sophisticated attackers do not rely on a single vulnerability. They chain multiple weaknesses to progress from reconnaissance to financial or operational impact.*

### The API Attack Chain

```
  1. DISCOVERY
     Find API endpoints via: mobile app decompilation, public docs,
     error messages, API enumeration, Google dorks, developer forums
                          │
                          ▼
  2. ENUMERATION
     Map API structure: endpoint paths, parameters, object IDs,
     version numbers, response patterns
                          │
                          ▼
  3. AUTHENTICATION ABUSE
     Obtain valid credentials: credential stuffing, token theft,
     leaked API keys, weak token validation
                          │
                          ▼
  4. AUTHORIZATION BYPASS
     BOLA — access other customers' objects
     BFLA — invoke admin functions
     Object property manipulation
                          │
                          ▼
  5. DATA ACCESS
     Extract sensitive data: customer PII, account data,
     transaction history, internal IDs
                          │
                          ▼
  6. BUSINESS LOGIC ABUSE
     Abuse legitimate functions: transfer velocity manipulation,
     workflow sequence abuse, threshold evasion
                          │
                          ▼
  7. AUTOMATION
     Scale the attack: automated scripts, bots,
     distributed attack infrastructure
                          │
                          ▼
  8. FINANCIAL / OPERATIONAL IMPACT
     Fraud, data theft, account takeover,
     operational disruption, regulatory breach
```

### Key Insight for Detection Teams
Attackers in Stages 1–3 may appear as normal traffic. Detectable signals typically emerge in Stages 3–5:
- Unusual object ID patterns
- Authorization failure spikes
- Abnormal request sequences
- Geographic anomalies
- Volume deviations

---

## Slide 09 — API Security vs Traditional Web Security

### Executive Takeaway
> *Securing the web or mobile frontend does not automatically secure the API backend. APIs require dedicated security controls, testing approaches, and monitoring strategies.*

### Comparison Matrix

| Dimension | Traditional Web Security | API Security |
|---|---|---|
| **Attack surface** | Web pages, forms, cookies | API endpoints, parameters, object IDs, request bodies |
| **Authentication** | Session cookies, web login | Tokens (JWT/OAuth), API keys, mTLS, short-lived credentials |
| **Authorization** | Page-level access control | Object-level, function-level, field-level authorisation per request |
| **Data exposure** | HTML rendered data | Full data objects in JSON/XML — often more than the UI displays |
| **Automation** | Limited by browser behaviour | Fully automatable at machine speed — thousands of requests/second |
| **Business logic** | Enforced partly in frontend | Must be entirely enforced in API backend |
| **Rate limiting** | Web server / CDN | Must be explicit per API endpoint and per consumer |
| **Discovery** | URL crawling | API enumeration, documentation leakage, mobile app decompilation |
| **Communication** | Browser-to-server | Machine-to-machine — no human in the loop |
| **Third-party** | Embedded scripts | Entire API integrations with external systems |
| **Monitoring** | Web access logs | API telemetry, token analytics, object-access patterns |
| **WAF coverage** | High — designed for web | Partial — API traffic may bypass web-specific rules |

### Critical Point for Architecture Teams

> Securing the **mobile app** or **web portal** does not protect the **API backend**. An attacker can bypass the frontend entirely and communicate directly with the API using standard tools such as curl, Postman, or custom scripts.

---

## Slide 10 — The API Estate Problem

### Executive Takeaway
> *"You cannot secure what you cannot see." Without a comprehensive API inventory, the bank cannot assess, monitor, or govern its API risk.*

### The Core Problem

Most organisations discover they have **significantly more APIs than they were aware of**, including:

```
  KNOWN APIs                        UNKNOWN / SHADOW APIs
  ──────────────────────────────    ──────────────────────────────────
  Production APIs (documented)      Deprecated API versions (v1, v2)
  Gateway-managed APIs              Development / test APIs (never removed)
  Formally approved APIs            Direct-to-backend APIs (bypassing gateway)
                                    Legacy system APIs
                                    Developer-created APIs
                                    Third-party APIs
                                    Cloud-native APIs
                                    Unmanaged microservice APIs
                                    Partner-exposed APIs
```

### Questions the Bank Must Be Able to Answer

1. How many APIs does the bank operate?
2. Where are they? Which are internet-facing?
3. Who owns each API?
4. What business process does each API support?
5. What data does each API process?
6. Who can access each API?
7. What can each API access downstream?
8. Which APIs are deprecated but still accessible?
9. Which APIs are not in the formal inventory?
10. Which APIs have known vulnerabilities?
11. Which APIs have never been security-tested?

### The Risk Consequence

An API that is unknown:
- Cannot be monitored
- Cannot be security-tested
- Cannot be patched
- Cannot be included in incident response
- Cannot be governed

> **An unknown, internet-facing, legacy API with weak authentication represents an unmanaged pathway into the bank's systems.**

---

## Slide 11 — Enterprise API Inventory Model

### Executive Takeaway
> *API inventory is a risk control, not merely a developer document. Every API the bank operates must be formally registered, owned, classified, and assessed.*

### Minimum Required Inventory Fields

| Category | Fields |
|---|---|
| **Identity** | API name, API ID, version, description |
| **Ownership** | Business owner, technical owner, development team |
| **Purpose** | Business process supported, consumer applications |
| **Classification** | Internal / External, Data classification, Criticality |
| **Exposure** | Internet-facing (Y/N), External consumers, Third-party dependency |
| **Security** | Authentication method, Authorisation mechanism, TLS enforced (Y/N) |
| **Environment** | Production / UAT / Dev / DR, Deployment environment |
| **Connectivity** | API endpoint(s), Downstream systems, Upstream consumers |
| **Risk** | Risk rating, Regulatory relevance, Financial transaction capable (Y/N) |
| **Assessment** | Last security test date, VAPT status, Known vulnerabilities |
| **Lifecycle** | Status (Active/Deprecated/Retired), Deprecation date, Retirement date |
| **Monitoring** | Included in SOC monitoring (Y/N), SIEM integration (Y/N) |

### Governance Requirement

The API inventory should be:
- **Formally owned** by IT Risk or a designated API Governance function
- **Mandatory** for all APIs before production deployment
- **Reviewed** at least annually and on material change
- **Integrated** with the bank's risk register and technology asset register
- **Automated** where possible through API discovery tooling

---

## Slide 12 — Secure API Lifecycle

### Executive Takeaway
> *Security must be embedded at every stage of the API lifecycle — from design through retirement. Security reviews at deployment are too late to address design-level weaknesses.*

### The Secure API Lifecycle

```
  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  PLAN    │ → │  DESIGN  │ → │  BUILD   │ → │   TEST   │ → │  DEPLOY  │
  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
       │               │               │               │               │
  API register    Threat model    Secure coding    SAST / DAST    API Gateway
  Business case   Auth design     Secrets mgmt     VAPT           TLS / Auth
  Data classify   Authz design    Input validation  Auth test      Rate limit
  Risk assess     Abuse cases     Dependency scan   Logic test     Logging
                                                                       │
  ┌──────────┐   ┌──────────┐   ┌──────────┐                         │
  │  RETIRE  │ ← │  REVIEW  │ ← │ OPERATE  │ ←───────────────────────┘
  └──────────┘   └──────────┘   └──────────┘
       │               │               │
  Disable endpts  Periodic VAPT   Continuous monitor
  Revoke creds    Auth/authz review   Anomaly detect
  Remove routes   Risk reassess   Vuln management
  Confirm migration  Inventory update  Access review
  DNS cleanup     Policy review   Cert/key rotation
```

### Security Controls by Stage

**Design Phase (critical — cheapest to fix here)**
- Threat modelling using STRIDE or PASTA methodology
- Authentication design (OAuth 2.0, mTLS, API key policy)
- Authorisation design (RBAC, ABAC, object-level authorisation)
- Data classification and minimisation strategy
- Abuse case analysis

**Build Phase**
- Secure coding standards for API development
- Secrets management — no hardcoded credentials
- Input validation and output encoding
- Dependency / third-party library scanning (SCA)

**Test Phase**
- SAST (Static Application Security Testing)
- DAST (Dynamic Application Security Testing)
- API-specific security testing (OWASP API Top 10)
- Authorisation and privilege testing
- Business logic abuse testing
- Fuzzing (malformed input testing)

**Deploy Phase**
- API Gateway configuration (authentication, rate limiting, logging)
- TLS enforcement (no unencrypted API traffic)
- WAF/WAAP integration
- Centralised logging connected to SIEM

**Operate Phase**
- Continuous monitoring and alerting
- Periodic vulnerability assessments
- Access and consumer reviews
- Certificate and key rotation schedule

**Retire Phase (often overlooked)**
- Formal disable of all endpoints
- Credential revocation
- Route removal from API Gateway
- Documentation removal or archival
- DNS and infrastructure cleanup
- Confirmation that all consumers have migrated

---

## Slide 13 — Enterprise API Security Architecture

### Executive Takeaway
> *A layered API security architecture ensures that no single control failure results in a breach. Defence-in-depth principles apply directly to API protection.*

### Reference Architecture

```
  ═══════════════════════════════════════════════════════════════════
                      EXTERNAL CONSUMERS
         Mobile Apps | Web Portal | Fintech Partners | Open Banking
  ═══════════════════════════════════════════════════════════════════
                              │
                    ┌─────────▼──────────┐
                    │  CDN / DDoS         │  ← Traffic scrubbing
                    │  Protection Layer   │     Geo-blocking / Bot mitigation
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │  WAF / WAAP         │  ← Web + API firewall
                    │  (API Protection)   │     OWASP rule sets
                    └─────────┬──────────┘     API schema validation
                              │
                    ┌─────────▼──────────┐
                    │  API GATEWAY        │  ← Authentication enforcement
                    │                     │     Rate limiting / Quotas
                    └─────────┬──────────┘     TLS termination / Logging
                              │
                    ┌─────────▼──────────┐
                    │  IDENTITY &         │  ← OAuth 2.0 / OIDC
                    │  AUTHENTICATION     │     MFA / Token management
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │  AUTHORISATION /    │  ← Policy enforcement (RBAC/ABAC)
                    │  POLICY ENGINE      │     Object-level access checks
                    └─────────┬──────────┘
                              │
         ┌───────────────────┬┴────────────────────┐
         │                   │                     │
  ┌──────▼──────┐   ┌────────▼───────┐   ┌────────▼────────┐
  │  PAYMENT    │   │  BANKING       │   │  PARTNER        │
  │  API        │   │  API SERVICES  │   │  API SERVICES   │
  └──────┬──────┘   └────────┬───────┘   └────────┬────────┘
         └──────────────────┬┴────────────────────┘
                            │
                  ┌─────────▼──────────┐
                  │  CORE SYSTEMS       │  ← Core banking / Payments
                  │  & DATABASES        │     AML / Fraud / Card data
                  └────────────────────┘

  ═══════════════════════════════════════════════
      HORIZONTAL SECURITY SERVICES (All Layers)
  ═══════════════════════════════════════════════
  SIEM / SOC Monitoring        API Discovery & Inventory
  Secrets Management           Identity & Access Management (IAM)
  Privileged Access (PAM)      Vulnerability Management
  Fraud Monitoring             Data Loss Prevention (DLP)
  Threat Intelligence          Continuous Security Testing
```

### Layer Security Responsibilities

| Layer | Primary Security Responsibility |
|---|---|
| CDN / DDoS | Traffic volume attacks, bot filtering, geographic controls |
| WAF / WAAP | Protocol attacks, OWASP Top 10, API schema enforcement |
| API Gateway | Authentication, rate limiting, TLS, routing, logging |
| Identity / Auth | Token issuance, MFA, session management |
| Authorisation | Per-request access decisions, policy enforcement |
| API Services | Business logic controls, input validation |
| Core Systems | Data access controls, encryption at rest |

> **No single layer is sufficient on its own. Each layer assumes the others may be partially bypassed or fail.**

---

## Slide 14 — Authentication vs Authorization

### Executive Takeaway
> *These two controls are distinct, both critical, and must be explicitly designed for every API. Authentication without proper authorisation is insufficient.*

### The Critical Distinction

```
  ┌────────────────────────────────────────────────────────────┐
  │                                                            │
  │  AUTHENTICATION               AUTHORISATION               │
  │  ══════════════               ═════════════               │
  │                                                            │
  │  "Who are you?"               "What are you               │
  │                                allowed to do?"            │
  │                                                            │
  │  Verifies IDENTITY            Verifies PERMISSION         │
  │                                                            │
  │  Precondition for             Must be enforced on         │
  │  API access                   EVERY API request           │
  │                                                            │
  └────────────────────────────────────────────────────────────┘
```

### Banking Examples

| Scenario | Authentication Result | Authorisation Result |
|---|---|---|
| Customer views own account | Customer logged in ✅ | This customer owns the account ✅ — allow |
| Customer views another account | Customer logged in ✅ | Not this customer's account ❌ — deny |
| Customer calls admin function | Customer logged in ✅ | Not an admin role ❌ — deny |
| Partner API access | Partner token valid ✅ | Partner authorised for this function? — must check |
| Internal service call | Service credential valid ✅ | Does this service need this data? — must check |

### Why a Valid Login Is Not Sufficient

A valid token confirms **who** made the request. It does **not** automatically establish:
- Whether the user owns the resource they are requesting
- Whether the user's role permits the function they are invoking
- Whether the request volume is within expected parameters

### The Most Common API Vulnerability

**BOLA (Broken Object-Level Authorization)** — consistently the most reported API vulnerability — arises precisely because teams implement authentication correctly but fail to verify object-level authorisation.

---

## Slide 15 — Zero Trust Principles for APIs

### Executive Takeaway
> *Zero Trust means no API consumer — internal or external — is implicitly trusted. Every request must be explicitly verified, authorised with least privilege, and continuously monitored.*

### Zero Trust for APIs — Core Principles

| Principle | API Application |
|---|---|
| **Verify explicitly** | Authenticate every request — no implicit trust for internal callers |
| **Least privilege** | Grant the minimum API access required for the specific function |
| **Assume breach** | Design systems assuming an attacker may have a valid credential |
| **Continuous verification** | Re-evaluate trust at each API request, not just at login |
| **Segment services** | Internal APIs must be network-segmented — not universally accessible |
| **Monitor continuously** | All API traffic must generate telemetry for anomaly detection |

### Key Technologies and Mechanisms

| Mechanism | Purpose | Appropriate Use |
|---|---|---|
| **OAuth 2.0** | Delegated authorisation framework | Customer-facing and partner APIs |
| **OpenID Connect** | Identity layer on top of OAuth 2.0 | Customer authentication |
| **JWT (JSON Web Tokens)** | Signed, stateless tokens | Short-lived API access tokens |
| **mTLS (Mutual TLS)** | Both parties authenticate via certificate | Service-to-service / machine-to-machine APIs |
| **Short-lived tokens** | Tokens expire quickly — reduces window of abuse | All API contexts |
| **Service identities** | Cryptographic workload identity | Cloud-native and microservice APIs |
| **API keys** | Simple shared secret | Lower-sensitivity partner integrations (with rotation policy) |

> No single authentication mechanism is appropriate for all API contexts. The bank should define a **tiered authentication policy** based on API risk classification.

### Speaker Notes
- mTLS is appropriate for internal service-to-service communication where both parties can hold certificates.
- OAuth 2.0 with short-lived JWTs is the industry standard for customer-facing API authentication.
- API keys alone are not suitable for high-value or sensitive APIs.
- Source: NIST SP 800-207 (Zero Trust Architecture), 2020.

---

## Slide 16 — API Monitoring and Detection

### Executive Takeaway
> *API abuse is often only detectable through behavioural analysis of API telemetry. Traditional network monitoring alone is insufficient. SOC visibility into API activity is essential.*

### What the Bank Should Monitor

**Authentication & Access**
- Authentication failures by endpoint, IP, and user
- Token anomalies (reuse, geographic inconsistency, unexpected expiry)
- API key usage from unusual locations or volumes

**Authorisation & Access Patterns**
- Authorization failures — particularly BOLA patterns (sequential object ID requests)
- Repeated access to high-sensitivity endpoints
- Privilege escalation attempts (function-level authorization failures)

**Volume & Velocity**
- Abnormal request volumes per consumer, endpoint, or IP
- Rate-limit violations
- API enumeration patterns (sequential parameter scanning)
- Error-rate spikes

**Behavioural Anomalies**
- Unusual geographic access patterns
- Access time anomalies
- Deprecated API usage from external IPs
- New, undiscovered API endpoints appearing in traffic

**Business Logic**
- Unusual transaction sequences
- Repeated small transfers to new beneficiaries
- Bulk data export patterns

### SOC Integration

```
  API Gateway Logs
  API Service Logs    →  Log Aggregation  →  SIEM  →  SOC Analysts
  Auth Service Logs                            │
  WAF / WAAP Logs                              ▼
                                      Detection Rules
                                      Anomaly Models
                                      Alert Triage
                                      Incident Response
```

### Detection Rule Examples

| Condition | Possible Signal |
|---|---|
| >50 authorization failures from one IP in 60 seconds | BOLA enumeration attempt |
| Authentication failure then success from two countries within 5 minutes | Account takeover / session hijacking |
| Sequential account ID increments in API requests | Object enumeration |
| API response size 10× normal for a given endpoint | Excessive data exposure |
| Calls to deprecated API version from external IP | Shadow API exploitation |

---

## Slide 17 — API Security and Fraud

### Executive Takeaway
> *Fraud teams and cybersecurity teams must share API telemetry. Attackers exploit the gap between these two disciplines to conduct automated, API-enabled fraud.*

### How APIs Enable Fraud

| Fraud Type | API Abuse Mechanism |
|---|---|
| **Account Takeover** | Credential stuffing attacks against authentication APIs; token theft |
| **Enumeration / Reconnaissance** | API calls to verify which account numbers, phone numbers, or emails are valid |
| **Synthetic Identity Fraud** | Abuse of onboarding/KYC APIs with fabricated identities |
| **Transaction Manipulation** | Business logic abuse in payment or transfer APIs |
| **Automated Payment Fraud** | High-volume automated API calls to initiate small transfers below thresholds |
| **Card Fraud** | Brute-force of card verification APIs (CVV, balance enquiry) |
| **Data Harvesting** | Bulk customer data extraction for downstream fraud use |

### The Correlation Challenge

Effective detection requires correlating signals across:

```
  IDENTITY                API ACTIVITY              TRANSACTION
  ─────────────────       ──────────────────────    ─────────────────
  Device fingerprint  +   API call patterns      +  Transaction type
  Authentication event    Request velocity           Amount
  Geographic location     Endpoint accessed          Beneficiary
  Session context         Object IDs accessed        Time of day
  Biometric signals       Error patterns             Channel
```

> **Cybersecurity and Fraud must share API telemetry.** An attack pattern that is invisible to either team alone may be clearly visible when signals are combined.

---

## Slide 18 — Third-Party and Partner API Risk

### Executive Takeaway
> *Third-party APIs extend the bank's attack surface beyond its direct control. Vendor security standards and contractual obligations must address API security explicitly.*

### The Third-Party API Risk Surface

**APIs the bank exposes to third parties:**
Open Banking TPPs, Fintech partners, Payment aggregators, Merchant APIs, White-label banking partners

**APIs the bank consumes from third parties:**
KYC / identity verification vendors, Payment network APIs, Cloud provider APIs, Credit bureau APIs, Fraud scoring APIs, Government system APIs

### Risk in Both Directions

```
  Third Party CONSUMES bank API     Bank CONSUMES third-party API
  ─────────────────────────────     ─────────────────────────────
  Credential abuse                  Compromised vendor API
  Unauthorised data access          Malicious / tampered response
  Rate limit exhaustion             Data integrity breach
  Reconnaissance                    Man-in-the-middle
  Privilege escalation              Dependency vulnerability
```

### Third-Party API Security Requirements

**Due Diligence**
- API security assessment as part of vendor onboarding
- Review of vendor API security architecture
- Evidence of penetration testing and vulnerability management

**Contractual Requirements**
- Mandatory TLS for all API communications
- Minimum authentication standards
- Incident notification obligations
- Right to audit API security controls
- Data minimisation requirements

**Operational Controls**
- Dedicated API credentials per third party
- Granular authorisation — third party accesses only what it needs
- Rate limiting and quotas per third-party consumer
- Regular access reviews

---

## Slide 19 — API Security Governance Framework

### Executive Takeaway
> *API security governance requires clear accountability across people, processes, and technology. Governance gaps are as dangerous as technical vulnerabilities.*

### Governance Framework

```
  PEOPLE                      PROCESS                     TECHNOLOGY
  ────────────────────        ──────────────────────────  ─────────────────────
  API Owner                   API Registration            API Gateway
  Business Owner              Risk Assessment             API Discovery Tool
  Security Architect          Security Review Gate        SIEM / SOAR
  CISO / IT Risk              Change Management           WAF / WAAP
  Development Teams           VAPT Programme              IAM Platform
  SOC                         Access Review               Secrets Management
  Compliance                  Incident Response Plan      Vuln Management
  Internal Audit              Lifecycle Management        API Security Platform
  Third-Party Risk            Reporting & Escalation      Log Management
```

### Governance Processes

| Process | Frequency | Owner |
|---|---|---|
| API registration (new APIs) | At development | Development / IT |
| Security review gate | Before deployment | Security Architecture |
| API risk assessment | Annually + on change | IT Risk |
| VAPT (critical APIs) | Annually minimum | Security / VAPT Team |
| API inventory reconciliation | Quarterly | IT Risk / API Governance |
| Third-party API review | Annually + on change | Third-Party Risk |
| Access review | Semi-annually | IT / Business Owner |
| API security KPI / KRI reporting | Monthly / Quarterly | CISO / IT Risk |

### Escalation Path

```
  API security finding → API Owner → Security Architecture → IT Risk
  → CISO (if material) → CRO (if risk appetite breach) → Board / Risk Committee
```

---

## Slide 20 — API Risk Classification Model

### Executive Takeaway
> *Not all APIs carry equal risk. A risk classification model ensures that the most critical APIs receive the most rigorous security controls and oversight.*

### Risk Classification Factors

| Factor | Higher Risk | Lower Risk |
|---|---|---|
| **Data sensitivity** | PII, financial, authentication data | Non-sensitive, public data |
| **Business criticality** | Core banking, payments, card | Non-critical business functions |
| **Financial transaction capability** | Can initiate or modify transactions | Read-only, non-financial |
| **Internet exposure** | Externally accessible | Internal only |
| **Consumer scope** | Public / many third parties | Single internal consumer |
| **Privilege level** | Administrative functions | Standard user functions |
| **Authentication strength** | Weak (API key only) | Strong (mTLS + OAuth) |
| **Regulatory impact** | Affects PCI, data protection, Open Banking | No direct regulatory scope |
| **Downstream system access** | Core banking, payment systems | Peripheral systems |

### Risk Classification Examples

| Risk Level | Example APIs | Required Controls |
|---|---|---|
| **Critical** | Payment initiation, Core banking data, Authentication, Card authorisation | mTLS + OAuth, annual VAPT, SOC monitoring, board-level reporting |
| **High** | Account balance, Customer profile, Open Banking APIs, Fraud scoring | OAuth, 6-monthly VAPT, SIEM alerting |
| **Medium** | Branch locator, Product information, Non-sensitive partner APIs | API key + TLS, annual review, standard monitoring |
| **Low** | Public rate API, Marketing content API | TLS, documentation only |

> [!IMPORTANT]
> The bank should define its own formal risk criteria and thresholds through a documented API Risk Classification Policy. Examples above are illustrative frameworks, not prescribed classifications.

---

## Slide 21 — API Security Maturity Model

### Executive Takeaway
> *Understanding current API security maturity enables the bank to prioritise investment, set a target state, and measure progress systematically.*

### Five-Level API Security Maturity Model

```
  Level 5 │ OPTIMISED   Continuous automated risk management, behavioral
           │             analytics, threat intelligence integration,
           │             policy-as-code, enterprise risk integration
           │
  Level 4 │ MANAGED     Continuous discovery, risk-based testing,
           │             centralized monitoring, full lifecycle
           │             management, third-party controls mature
           │
  Level 3 │ CONTROLLED  Security testing programme, authentication
           │             and authorisation standards enforced,
           │             SOC integration, governance established
           │
  Level 2 │ DOCUMENTED  API inventory exists, basic security
           │             standards documented, some testing,
           │             ownership assigned for major APIs
           │
  Level 1 │ UNKNOWN     No reliable API inventory, no governance,
           │             no consistent security controls, no
           │             formal API security ownership
```

### Maturity Level Characteristics

| Level | Inventory | Standards | Testing | Monitoring | Governance |
|---|---|---|---|---|---|
| **1 — Unknown** | None | None | Ad hoc | None | None |
| **2 — Documented** | Partial | Basic | Occasional | Basic logs | Informal |
| **3 — Controlled** | Comprehensive | Enforced | Structured | SIEM integrated | Formal process |
| **4 — Managed** | Automated | Policy-driven | Continuous | Behavioural analytics | Enterprise risk |
| **5 — Optimised** | Real-time | Automated enforcement | Continuous + threat intel | AI-assisted | Board-level KRI |

> [!NOTE]
> This model should be used as a **self-assessment framework**. The bank's current maturity should be formally assessed by IT Risk or Internal Audit before determining target state and investment priorities.

---

## Slide 22 — Banking Incident Case Studies

### Executive Takeaway
> *Documented API-related incidents across financial services and technology demonstrate that API vulnerabilities translate to material business impact. These are not theoretical risks.*

> [!NOTE]
> All incidents described are based on publicly available reporting and post-incident disclosures. Sources are cited. Facts are distinguished from analysis.

---

### Case Study 1 — Optus (Australia) — 2022

**What happened:** In September 2022, Optus suffered a significant data breach affecting approximately 9.8 million customers.

**Attack vector:** Attackers accessed a publicly exposed API that did not require authentication. The API had been inadvertently exposed to the internet.

**API / Security weakness:**
- API endpoint exposed to the internet without authentication
- API allowed sequential enumeration of customer records

**What was exposed:** Names, dates of birth, phone numbers, email addresses, physical addresses, passport and driving licence numbers.

**Business impact:** Regulatory investigation by the Australian Information Commissioner; significant reputational damage; CEO resignation; regulatory reform resulting in new mandatory data breach notification requirements.

**Root cause:** Insufficient API inventory management — the exposed endpoint was not identified as internet-facing and was not subject to the same controls as known production APIs (Shadow API pattern).

**Applicable Controls:** API inventory management, authentication, API discovery tooling, internet-facing API review.

**Sources:** Australian Information Commissioner investigation announcement (November 2022); Australian Federal Police media statement.

---

### Case Study 2 — Peloton — 2021

**What happened:** Security researcher Jan Masters (reported by TechCrunch, May 2021) discovered that Peloton's API exposed private user data without authentication.

**Attack vector:** API endpoints accessible without authentication tokens. Any caller could query user data by user ID.

**API / Security weakness:**
- Missing authentication on API endpoints
- No rate limiting on data queries
- Broken Object-Level Authorization

**What was exposed:** User ages, locations, workout data, profile information — including data of users who had set their profiles to "private."

**Business impact:** Public disclosure, reputational damage, regulatory attention; Peloton initially disputed the severity before acknowledging and patching the issue.

**Root cause:** Authentication not enforced consistently across all API endpoints; privacy settings enforced at the UI layer but not at the API layer.

**Security lessons:**
- API security must be enforced at the API layer — not assumed from UI controls
- Privacy settings must be reflected in API authorisation, not just frontend rendering

---

### Case Study 3 — T-Mobile — 2023

**What happened:** T-Mobile disclosed a breach in January 2023 affecting approximately 37 million customers.

**Attack vector:** Attackers accessed an API to query and extract customer account data over approximately six weeks before detection.

**API / Security weakness:**
- API exposed customer data subject to bulk enumeration
- Six-week dwell time indicates insufficient API monitoring and anomaly detection

**What was exposed:** Names, billing addresses, email addresses, phone numbers, dates of birth, account numbers, plan details.

**Business impact:** FCC investigation; $60M settlement (announced 2024); seventh major breach at T-Mobile since 2018.

**Root cause:** API not adequately protected against bulk enumeration; insufficient monitoring to detect sustained abnormal query volumes.

**Security lessons:**
- API monitoring must detect sustained data extraction over extended periods
- Rate limiting and anomaly detection are essential for data-bearing APIs
- SOC detection rules must include API enumeration patterns

**Sources:** T-Mobile SEC Form 8-K (January 2023); FCC consent decree documentation (2024).

---

### Case Study 4 — Latitude Financial — 2023

**What happened:** Australian fintech Latitude Financial disclosed a data breach in March 2023 affecting approximately 14 million customers across Australia and New Zealand.

**Attack vector:** Attackers used stolen employee credentials to access systems, including identity document data processed through KYC and onboarding integrations.

**Relevance to API security:** Illustrates how credential compromise combined with API access to sensitive data repositories can result in large-scale data exposure. Internal API access granted to the compromised credential enabled mass extraction.

**Business impact:** Largest known personal data breach in Australian financial services history; significant regulatory scrutiny; class action proceedings.

**Security lessons:**
- Privileged access to sensitive data-bearing APIs must be subject to MFA and PAM controls
- Internal APIs accessing sensitive data must be monitored
- API access logs must be retained and anomaly-monitored

**Sources:** Latitude Financial ASX disclosure statements (March–April 2023); Australian Government CISC advisory.

---

## Slide 23 — Regulatory and Framework Mapping

### Executive Takeaway
> *API security controls align with multiple regulatory and standards frameworks. Compliance obligations and industry best practice converge on the same fundamental controls.*

> [!WARNING]
> Regulatory applicability varies by jurisdiction, business model, and licensing type. This mapping is provided as a reference framework. The bank's Compliance team must confirm which regulations apply and at what standard.

### Framework Mapping

| Framework | Relevance to API Security | Key Controls |
|---|---|---|
| **NIST Cybersecurity Framework (CSF 2.0)** | Identify, Protect, Detect, Respond, Recover functions map directly to API security lifecycle | Asset inventory (Identify), Access controls (Protect), Monitoring (Detect) |
| **NIST SP 800-204 series** | Specifically addresses microservices and API security architecture | API gateway controls, service mesh security, authentication |
| **NIST SP 800-53 Rev 5** | Comprehensive control catalogue covering API-relevant domains | AC (Access Control), AU (Audit), SC (System & Comms), SI (System Integrity) |
| **OWASP API Security Top 10 (2023)** | De facto API security testing and design standard | All 10 risk categories |
| **ISO/IEC 27001:2022** | Information security management system standard | A.8 Technology controls |
| **ISO/IEC 27002:2022** | Control guidance for secure development and access controls | Clause 8.20, 8.25, 8.29 |
| **PCI DSS v4.0** | Applicable to APIs processing cardholder data | Req 6 (Secure systems), Req 7 (Access control), Req 10 (Logging), Req 11 (Testing) |
| **Open Banking / PSD2 / equivalents** | Mandates API exposure to TPPs — security must meet regulatory standards | Strong Customer Authentication, TPP authorisation |
| **DORA (EU)** | ICT risk management, third-party risk, digital resilience | API risk in ICT risk framework |
| **Data Protection / Privacy Law** | APIs processing personal data must comply with data minimisation, purpose limitation | Response filtering, data classification, third-party data flows |
| **CISA Secure by Design** | Build security into software by design | Threat modelling, authentication design, secure defaults |

### Key Distinction

| Mandatory Regulatory Requirement | Industry Best Practice |
|---|---|
| Data protection obligations for APIs processing personal data | OWASP API Security Top 10 |
| PCI DSS for card-data-bearing APIs | NIST SP 800-204 API architecture guidance |
| Open Banking API security standards | API maturity model assessment |
| DORA third-party ICT risk (EU) | API discovery tooling |

---

## Slide 24 — Executive KPIs and KRIs

### Executive Takeaway
> *What gets measured gets managed. API security requires a formal metrics framework to track performance (KPIs) and signal emerging risk (KRIs).*

### Key Performance Indicators (KPIs)
*Measure programme effectiveness and control implementation*

| KPI | Target | Reporting Frequency |
|---|---|---|
| % of APIs formally registered in inventory | 100% of production APIs | Monthly |
| % of APIs with named business and technical owner | 100% | Monthly |
| % of critical/high APIs assessed (VAPT) in last 12 months | 100% | Quarterly |
| % of internet-facing APIs with current security assessment | 100% | Quarterly |
| % of APIs using approved authentication standard | 100% (critical), 95%+ (high) | Quarterly |
| % of APIs integrated with centralised monitoring / SIEM | 100% (critical/high) | Monthly |
| % of deprecated APIs confirmed disabled | 100% | Monthly |
| % of third-party APIs with completed security assessment | 100% (critical partners) | Semi-annually |
| Mean time to remediate critical API vulnerabilities | ≤30 days | Monthly |
| Mean time to remediate high API vulnerabilities | ≤90 days | Monthly |

### Key Risk Indicators (KRIs)
*Signal risk deterioration — trigger management attention when thresholds exceeded*

| KRI | Threshold (Illustrative) | Escalation |
|---|---|---|
| Number of unregistered APIs discovered | >0 internet-facing unregistered | CISO + IT Risk |
| Number of deprecated APIs still active and internet-facing | >0 | CISO |
| Number of critical API vulnerabilities unremediated >30 days | >0 | CRO |
| Number of API-related security incidents | Any confirmed incident | CISO + CRO |
| Number of unauthorized API access events detected | Any confirmed event | CISO + Fraud |
| API authentication failure rate spike (>3× baseline) | Threshold breach | SOC + CISO |
| Third-party API breach affecting the bank's data | Any event | CRO + Board |
| APIs with no security assessment in 24+ months | >5% of critical estate | IT Risk |

> [!NOTE]
> Thresholds shown are illustrative. The bank should define formal risk appetite thresholds through its IT Risk Management and Risk Committee processes.

---

## Slide 25 — Recommended Roadmap

### Executive Takeaway
> *A six-phase roadmap transforms API security from an ad hoc technical concern into a managed, governed enterprise risk programme.*

### Phased Roadmap

```
  PHASE 1 │ DISCOVER (Months 1-3)
  ─────────┤ Build enterprise API inventory
           │ Identify all internet-facing APIs
           │ Identify critical and high-risk APIs
           │ Identify shadow/deprecated APIs
           │ Deploy API discovery tooling

  PHASE 2 │ ASSESS (Months 2-5)
  ─────────┤ Risk-classify all inventoried APIs
           │ API security architecture review
           │ Authentication and authorisation assessment
           │ VAPT on critical and high APIs
           │ Third-party API security review

  PHASE 3 │ PROTECT (Months 4-8)
  ─────────┤ Centralise critical APIs through API Gateway
           │ Enforce authentication standards (OAuth 2.0 / mTLS)
           │ Implement authorisation controls (object-level)
           │ Deploy rate limiting and quotas
           │ Implement secrets management
           │ Apply response minimisation / data filtering

  PHASE 4 │ DETECT (Months 6-10)
  ─────────┤ Centralise API logging
           │ Integrate API telemetry with SIEM
           │ Develop SOC detection rules for API threats
           │ Implement behavioural baseline monitoring
           │ Connect fraud and cybersecurity API signals

  PHASE 5 │ GOVERN (Months 8-14)
  ─────────┤ Publish API security policy and standards
           │ Establish API lifecycle governance process
           │ Implement security review gate for new APIs
           │ Define third-party API security requirements
           │ Establish KPI/KRI reporting to Risk Committee
           │ Launch API owner awareness programme

  PHASE 6 │ CONTINUOUSLY IMPROVE (Month 12+)
  ─────────┤ Automated API discovery and inventory reconciliation
           │ Continuous API security testing (CI/CD integration)
           │ Threat intelligence integration
           │ Behavioural analytics for API abuse detection
           │ API security maturity reassessment
```

---

## Slide 26 — Top 10 Actions for the Bank

### Executive Takeaway
> *Ten immediate and near-term actions to materially improve the bank's API security posture.*

| # | Action | Owner | Priority |
|---|---|---|---|
| **1** | **Build a complete API inventory** — Register every API, assign an owner, classify internet-facing and critical APIs | IT Risk + Development | Immediate |
| **2** | **Identify and disable all deprecated and shadow APIs** — Sweep all environments for non-current API versions, test APIs, and undocumented endpoints | IT + Security Architecture | Immediate |
| **3** | **Conduct security assessment (VAPT) on all internet-facing and critical APIs** — Prioritise payment, authentication, core banking, and Open Banking APIs | Security / VAPT Team | Immediate |
| **4** | **Enforce authentication standards** — Mandate OAuth 2.0/mTLS for all external and critical APIs; remove legacy API key-only authentication from high-risk APIs | Security Architecture + Dev | Short-term |
| **5** | **Implement and test object-level authorisation** — Every data API must verify the requesting user owns the object being requested | Development + Security | Short-term |
| **6** | **Integrate API telemetry into the SIEM/SOC** — All critical and high API logs must feed the SOC; develop detection rules for OWASP API Top 10 patterns | SOC + IT | Short-term |
| **7** | **Implement rate limiting on all external APIs** — Prevent automated abuse and denial-of-service; define limits per endpoint and per consumer | Infrastructure + Dev | Short-term |
| **8** | **Implement secrets management** — Eliminate all hardcoded API credentials; deploy a secrets management platform; implement pre-commit secret scanning | Development + Security | Short-term |
| **9** | **Establish API security governance** — Define formal API security policy, lifecycle process, security review gate, and ownership model | IT Risk + CISO | Medium-term |
| **10** | **Assess third-party API security** — Review all critical vendor and fintech API integrations; update contracts to include API security requirements | Third-Party Risk + Legal | Medium-term |

---

## Slide 27 — Final Executive Message

### The Core Message

> **APIs are now part of the bank's critical infrastructure.**

They are the mechanisms through which the bank serves customers digitally, processes financial transactions, connects with partners and fintechs, delivers on Open Banking obligations, and operates its internal systems.

---

### The Five Questions Every Executive Should Be Able to Answer

> **Do we know every API the bank operates?**

> **Do we know who can access each API?**

> **Do we know what each API can access?**

> **Do we know what data each API exposes?**

> **Can we detect abnormal API behaviour — and respond rapidly?**

---

### The Enterprise Risk Connection

```
  API Security
       │
       ├── CUSTOMER TRUST          Protecting customer data and accounts
       │
       ├── FINANCIAL PROTECTION    Preventing fraud and transaction abuse
       │
       ├── OPERATIONAL RESILIENCE  Maintaining service availability
       │
       ├── CYBERSECURITY           Defending the digital attack surface
       │
       └── REGULATORY COMPLIANCE   Meeting legal and regulatory obligations
```

---

### The Call to Action

API security requires:

- Enterprise governance — not just developer responsibility
- Formal inventory and ownership — not implicit assumptions
- Risk-based testing — not one-time, point-in-time review
- SOC detection — not reliance on perimeter controls alone
- Executive accountability — not delegation to IT

> **The bank that knows its APIs, owns them, tests them, monitors them, and governs them is the bank that manages this risk. The bank that does not is exposed to risks it may not even be able to see.**

---

## Annex A — IT Risk API Security Assessment Checklist

> Use this checklist when assessing new or existing APIs as part of the IT risk assessment or Internal Audit review process.

### API Identity & Ownership
- [ ] 1. Is the API formally registered in the enterprise API inventory?
- [ ] 2. Is there a named business owner?
- [ ] 3. Is there a named technical owner?
- [ ] 4. Is the API version documented?
- [ ] 5. What business process does the API support?

### Data & Exposure Classification
- [ ] 6. What categories of data does the API process? (PII, financial, authentication, internal system data)
- [ ] 7. Is the API internet-facing?
- [ ] 8. Who consumes the API? (Internal users, mobile app, third parties, partners)
- [ ] 9. Are all consumers formally authorised and documented?
- [ ] 10. What downstream systems does the API access?

### Authentication & Authorisation
- [ ] 11. What authentication mechanism is implemented? (OAuth 2.0, mTLS, API key, none)
- [ ] 12. Is the authentication mechanism appropriate for the API's risk classification?
- [ ] 13. How is object-level authorisation enforced?
- [ ] 14. How is function-level authorisation enforced?
- [ ] 15. Has object-level authorisation been explicitly tested (BOLA testing)?
- [ ] 16. Has function-level authorisation been explicitly tested (BFLA testing)?

### Technical Controls
- [ ] 17. Is rate limiting implemented? At what thresholds?
- [ ] 18. Are response payloads minimised to return only required data?
- [ ] 19. Is input validation implemented and tested?
- [ ] 20. Is TLS enforced for all API traffic?
- [ ] 21. Are secrets managed via an approved secrets management platform?

### Monitoring & Detection
- [ ] 22. Is API traffic logged? Where are logs stored and how long are they retained?
- [ ] 23. Is the API integrated with the SIEM / centralised monitoring platform?
- [ ] 24. Is the API included in SOC monitoring and detection rules?
- [ ] 25. Are authentication and authorisation failures alerted on?

### Security Testing
- [ ] 26. Has a VAPT been performed? When? By whom?
- [ ] 27. Are OWASP API Security Top 10 risks addressed in security testing?
- [ ] 28. Is security testing integrated into the CI/CD pipeline?
- [ ] 29. Are known vulnerabilities tracked and remediated within policy timescales?

### Lifecycle & Governance
- [ ] 30. Is the API lifecycle status documented? (Active / Deprecated / Retired)
- [ ] 31. If deprecated, has the endpoint been confirmed as disabled?
- [ ] 32. Are third-party integrations documented and assessed?
- [ ] 33. Is the API included in the incident response plan?
- [ ] 34. Is there evidence of a periodic security review?
- [ ] 35. Has the API owner confirmed their responsibilities?

---

## Annex B — API Governance RACI

| Activity | CISO/IT Risk | Security Architecture | Development | SOC | Business Owner | Compliance | Internal Audit |
|---|---|---|---|---|---|---|---|
| API inventory maintenance | A | C | R | I | R | I | I |
| Security review gate | A | R | C | I | C | I | — |
| Risk assessment | R | C | C | I | C | C | I |
| VAPT commissioning | A | R | C | I | I | I | I |
| SIEM/monitoring integration | A | C | C | R | I | — | I |
| Third-party API review | A | C | I | I | C | C | I |
| KPI/KRI reporting | R | C | C | C | C | C | I |
| Policy development | A | C | C | I | C | R | I |
| Incident response (API) | A | C | C | R | I | I | I |
| Internal audit review | I | C | C | I | I | C | R |

*R = Responsible | A = Accountable | C = Consulted | I = Informed*

---

## Annex C — References

### Primary Standards & Frameworks

1. **OWASP API Security Top 10 (2023 Edition)**
   https://owasp.org/API-Security/editions/2023/en/0x11-t10/

2. **NIST Cybersecurity Framework (CSF 2.0)** — February 2024
   https://doi.org/10.6028/NIST.CSWP.29

3. **NIST SP 800-204D: Strategies for Integration of Software Supply Chain Security in DevSecOps CI/CD Pipelines** (2023)
   https://csrc.nist.gov/publications/detail/sp/800-204d/final

4. **NIST SP 800-207: Zero Trust Architecture** — August 2020
   https://doi.org/10.6028/NIST.SP.800-207

5. **NIST SP 800-53 Rev 5: Security and Privacy Controls for Information Systems and Organizations**
   https://doi.org/10.6028/NIST.SP.800-53r5

6. **ISO/IEC 27001:2022** — Information security, cybersecurity and privacy protection — ISMS requirements
   https://www.iso.org/standard/27001

7. **ISO/IEC 27002:2022** — Information security controls
   https://www.iso.org/standard/75652.html

8. **PCI DSS v4.0.1** — Payment Card Industry Data Security Standard
   https://www.pcisecuritystandards.org/

9. **CISA Secure by Design Principles**
   https://www.cisa.gov/resources-tools/resources/secure-by-design

### Incident Reports & Regulatory Filings

10. **Australian Information Commissioner** — Optus Data Breach Investigation (2022)
    https://www.oaic.gov.au/

11. **T-Mobile SEC Form 8-K** — Disclosure of API Data Breach (January 2023)

12. **FCC Consent Decree — T-Mobile** (2024)

13. **Latitude Financial ASX Announcement** — Data Breach Disclosure (March 2023)

14. **TechCrunch** — "Peloton's Leaky API Exposes Riders' Private Account Data" (May 2021)

### Additional Resources

15. **CISA: API Security — Best Practices** (ongoing guidance)
    https://www.cisa.gov/

16. **European Banking Authority (EBA): Guidelines on ICT and Security Risk Management**
    https://www.eba.europa.eu/

17. **Financial Stability Board (FSB): Cyber Incident Reporting**
    https://www.fsb.org/

18. **NIST SP 800-204: Security Strategies for Microservices-based Application Systems**
    https://csrc.nist.gov/publications/detail/sp/800-204/final

---

*End of Presentation*

---

> [!NOTE]
> **Document Control**
>
> | Version | Date | Author | Status |
> |---|---|---|---|
> | 1.0 | 2026-Q4 | Information Security / IT Risk | Draft for Review |
>
> **Distribution:** Senior Management, Board Risk Committee, CISO, CTO, CRO, Internal Audit, Compliance, IT Risk Management
>
> **Classification:** CONFIDENTIAL — MANAGEMENT USE ONLY
>
> **Review Cycle:** Annual or on material change to API security landscape
