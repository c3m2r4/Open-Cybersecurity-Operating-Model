---
title: Template - Professional Finding (Full)
category: Template
tags:
  - Template
  - OffensiveSecurity
  - Reporting
date_created: 2026-08-24
status: MAINTAINED
---

# Template — Professional Finding (Full)

> **Instructions for use:** Use this template to document findings from offensive security assessments, vulnerability assessments, or red team engagements. All fields must be completed. Delete these instructions before finalizing.

---

## [Finding Title: e.g., Unconstrained Kerberos Delegation on Web Server]

**Finding ID:** [e.g., PT-2026-001]
**Severity:** [Critical / High / Medium / Low / Informational]
**Status:** [Open / Remediated / Risk Accepted]

### 1. Context and Risk

**Affected Asset(s):**
- [List specific hostnames, IP addresses, URLs, or components affected]

**Business Context:**
- [What business process does this asset support? e.g., "This server hosts the internal HR portal containing employee PII."]

**Threat:**
- [Who/what would realistically exploit this? e.g., "An attacker who has gained initial access to the internal network via phishing."]

**Impact:**
- [What is the business impact if exploited? e.g., "Allows the attacker to impersonate any domain user, including Domain Admins, leading to full domain compromise and access to all enterprise data."]

**Likelihood:**
- [How likely is exploitation? e.g., "High. The server is broadly accessible internally, and the exploitation tools are automated and widely available."]

**Risk Statement:**
- [Combine Threat, Vulnerability, and Impact. e.g., "A configuration weakness on the HR server allows an internal attacker to escalate privileges to domain administrator, resulting in a total compromise of the Active Directory environment."]

---

### 2. Technical Details

**Description (Vulnerability):**
- [Explain the technical flaw clearly. e.g., "The computer account for WEBSERVER01 has the 'Trust this computer for delegation to any service (Kerberos only)' attribute enabled. This is known as unconstrained delegation."]

**Attack Scenario:**
- [Step-by-step how it is exploited]
1. Attacker compromises a standard user workstation on the internal network.
2. Attacker coerces a high-privileged account (e.g., Domain Controller computer account) to authenticate to WEBSERVER01.
3. Because WEBSERVER01 has unconstrained delegation, the Domain Controller's Ticket Granting Ticket (TGT) is cached in WEBSERVER01's memory.
4. Attacker extracts the TGT from WEBSERVER01 memory and uses it to impersonate the Domain Controller.

**Evidence:**
- [Insert screenshots, code snippets, or command output demonstrating the vulnerability or exploit path. Highlight key areas.]

---

### 3. Controls and Root Cause

**Existing Controls & Failure:**
- [What controls were in place, and why did they fail? e.g., "Endpoint Detection (EDR) is installed, but did not prevent the credential extraction because it was running in audit-only mode on servers."]

**Root Cause:**
- [Configuration / Design / Process / Knowledge]
- [e.g., "Configuration. The server was misconfigured during legacy application deployment where exact delegation requirements were not understood."]

---

### 4. Remediation

**Recommendation:**
- [High-level recommendation. e.g., "Remove unconstrained delegation and replace with Resource-Based Constrained Delegation (RBCD) if delegation is required by the application."]

**Technical Remediation Steps:**
1. [Step 1: Open Active Directory Users and Computers.]
2. [Step 2: Navigate to WEBSERVER01 properties -> Delegation tab.]
3. [Step 3: Change setting to 'Do not trust this computer for delegation'.]
4. [Step 4: If application breaks, implement Constrained Delegation specific only to the required backend service.]

**Residual Risk:**
- [What risk remains after remediation? e.g., "Low. Removing unconstrained delegation removes this specific attack path."]

---

### 5. Retest and Closure

**Retest Criteria:**
- [How will we prove it is fixed? e.g., "Query Active Directory to confirm the userAccountControl attribute for WEBSERVER01 no longer contains the TRUSTED_FOR_DELEGATION flag."]

**References:**
- [Links to vendor documentation, CVEs, or internal policies]
- https://learn.microsoft.com/en-us/windows-server/security/kerberos/configuring-kerberos-delegation
