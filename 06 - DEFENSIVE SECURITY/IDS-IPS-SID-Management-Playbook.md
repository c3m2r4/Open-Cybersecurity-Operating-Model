# IDS/IPS SID Management & Threat Detection Playbook

**Document Owner:** Chief Information Security Officer (CISO)  
**Target Audience:** Security Operations Center (SOC) Analysts, Network Security Engineers  
**Classification:** Internal - Confidential  

---

## 1. Executive Summary & Core Philosophy

In a mature enterprise Security Operations Center (SOC), Intrusion Detection and Prevention Systems (IDS/IPS) are critical for identifying and mitigating threats. However, raw rule sets (such as Snort or Suricata feeds) are inherently noisy and often misaligned with specific organizational risk profiles. 

This playbook outlines the strategic and tactical management of Signature IDs (SIDs). Our philosophy is simple: **High Fidelity, Low Noise, Rapid Response.** 

We achieve this through systematic state management—explicitly controlling which rules are enabled, disabled, modified, or dropped—ensuring our analysts focus on actionable intelligence rather than alert fatigue.

---

## 2. SID State Management Architecture

Our IDS/IPS infrastructure relies on automated SID state management to dynamically apply tuning configurations whenever rule sets are updated. This is governed by four core configuration files.

### 2.1. `enablesid.conf` (Activation Layer)
Defines which security rules must be actively monitored. We prioritize enabling rules based on verifiable threat intelligence rather than broad ranges.

**Targeting Methods & Best Practices:**
- **Single SIDs & Ranges:** Use sparingly for known, critical coverage (e.g., `1:1034`, `1:220-1:3264`). Avoid enabling massive blocks indiscriminately.
- **CVE/Bugtraq Targeting:** Preferred for covering known exploited vulnerabilities (e.g., `cve:2021-44228` for Log4j).
- **PCRE (Regex):** Use for behavioral pattern matching (e.g., `pcre:(?i)ransomware`). Be cautious, as broad regex can cause high CPU load and false positives.
- **Vendor Categories:** Enable specific high-value feeds (e.g., `emerging-malware`, `etpro-cnc`).

### 2.2. `disablesid.conf` (Suppression Layer)
Silences noisy, low-value, or legacy rules. This is our primary tool for reducing SOC alert fatigue.

**Suppression Targets:**
- Legacy vulnerabilities with negligible risk in our hardened environment (e.g., MS00-MS02 exploits).
- False-positive-prone categories (e.g., generic `emerging-scan` rules on external-facing load balancers).
- Broad, overly sensitive PCRE matches.

### 2.3. `modifysid.conf` (Behavioral Tuning Layer)
Modifies the internal logic, targets, and variables of existing rules without changing their enabled/disabled state.

**Tuning Objectives:**
- **Insider Threat Visibility:** Change `$EXTERNAL_NET` to `any` on specific rules to ensure internal-to-internal lateral movement (e.g., compromised workstations attacking servers) is detected.
- **Port Context Hardening:** Expand coverage (e.g., mapping `"HTTP_PORTS"` to include `"HTTPS_PORTS"` or non-standard web ports).

### 2.4. `dropsid.conf` (Active Mitigation Layer)
Elevates rules from "Alert" to "Drop." When a rule in this list triggers, the packet is silently discarded.

**Usage Rules:**
- Only apply to high-confidence, low-false-positive signatures (e.g., known malware C2 beacons, verified exploit kits).
- Do not blanket-drop reconnaissance scans, as this can impede our ability to analyze the attacker's methodology.

---

## 3. Incident Response: Internal Reconnaissance

When IDS alerts trigger, context dictates severity. A critical scenario our SOC must immediately triage is suspected internal reconnaissance.

### Scenario: Internal Scanning Detection
**Alert:** `ET SCAN Possible Nmap User-Agent Observed (SID: 1:2024364)`
**Traffic:** Internal IP (e.g., 10.0.40.109) to Internal Server (e.g., 10.0.20.10)
**Supporting Evidence:** Repeated, high-frequency alerts, alongside `Applayer Mismatch` or `Unable to match response` logs.

### 3.1. Threat Assessment (Severity: HIGH)
External scans are background noise; internal scans signify that an attacker or unauthorized scanner is inside the network boundary (MITRE ATT&CK: **T1046 Network Service Discovery**).

### 3.2. Targeted Services of Concern
- **Port 135 (RPC):** Classic endpoint mapper enumeration.
- **Port 5985 (WinRM):** Targeted for lateral movement and remote execution.
- **Port 5357 (WSD):** Windows service discovery.

### 3.3. Immediate Action Plan
1. **Source Identification:** Identify the owner and purpose of the scanning IP. Is it an authorized vulnerability scanner (e.g., Nessus), a misconfigured IT tool, or a compromised endpoint?
2. **Endpoint Investigation:** Review EDR telemetry on the source machine for malicious processes, unauthorized scheduled tasks, or abnormal PowerShell executions.
3. **Containment:** If malicious, isolate the source host via EDR or switch-level port security.
4. **Target Hardening:** Verify that target servers have strict firewall rules (e.g., WinRM should only be accessible from dedicated management subnets, not general user VLANs).

---

## 4. Continuous Improvement

IDS/IPS tuning is an iterative process. The SOC must continuously:
1. Review top 10 noisiest alerts weekly.
2. Update `disablesid.conf` to suppress verified false positives.
3. Map new zero-day vulnerabilities (CVEs) into `enablesid.conf` rapidly.
4. Ensure lateral movement detection logic remains robust via `modifysid.conf`.

---

## Appendix: Reference Configurations

Below are production-ready baseline templates for the primary SID management files. Use these as starting points and tune to your specific environment.

### `enablesid.conf`

```conf
############################################################
# ENABLE SID CONFIGURATION - PRODUCTION SECURITY BASELINE #
############################################################

# -------------------------------
# 🔹 Core Critical Security Rules
# -------------------------------
1:1034,1:9837,1:1270,1:3390,1:710,1:1249,3:13010

# -------------------------------
# 🔹 High-risk attack detection ranges
# (Brute force, scanning, malware behavior)
# -------------------------------
1:220-1:3264
3:13010-3:13013

# -------------------------------
# 🔹 Microsoft vulnerability coverage
# Enable known exploited MS patches
# -------------------------------
MS09-008,MS10-002,MS12-020,MS15-034,MS17-010

# -------------------------------
# 🔹 CVE-based threat intelligence enablement
# Focus on actively exploited vulnerabilities
# -------------------------------
cve:2008-4250
cve:2017-0144
cve:2019-0708
cve:2021-44228
cve:2022-22965

# -------------------------------
# 🔹 Bugtraq advisory based rules
# -------------------------------
bugtraq:21301,bugtraq:43789

# -------------------------------
# 🔹 Web application attack detection (PCRE tuning)
# -------------------------------
pcre:SQLi
pcre:XSS
pcre:(?i)wordpress
pcre:(?i)joomla
pcre:(?i)phpmyadmin

# -------------------------------
# 🔹 Microsoft exploit pattern detection (PCRE advanced)
# -------------------------------
pcre:MS(0[7-9]|1[0-9]|2[0-6])-\d+
pcre:(?i)smb
pcre:(?i)ransomware

# -------------------------------
# 🔹 Network attack behavior patterns
# -------------------------------
pcre:(?i)portscan
pcre:(?i)bruteforce
pcre:(?i)botnet
pcre:(?i)command.*control

# -------------------------------
# 🔹 Vendor-specific rule sets
# -------------------------------
snort_web-iis
snort_netbios
emerging-malware
emerging-exploit
emerging-shellcode
etpro-ransomware
etpro-trojan
etpro-cnc

# -------------------------------
# 🔹 Safe enterprise baseline tuning
# (reduces noise while keeping coverage)
# -------------------------------
1:2000-1:2100
1:3000-1:3100

# -------------------------------
# 🔹 Explicit exclusions override note
# (handled in disablesid.conf usually, but documented here)
# -------------------------------
# 1:1011 # Example: false positive rule excluded elsewhere

############################################################
# END OF ENABLE SID CONFIGURATION
############################################################
```

### `modifysid.conf`

```conf
############################################################
# MODIFY SID CONFIGURATION - PRODUCTION IDS TUNING LAYER  #
############################################################

# --------------------------------------------------------
# 🔹 CORE PRINCIPLE
# This file ONLY modifies rule content (not enable/disable state)
# GID:1 rules only (standard Snort/Suricata rules)
# --------------------------------------------------------

# ========================================================
# 🔐 1. ATTACK SURFACE NORMALIZATION (EXTERNAL → INTERNAL)
# ========================================================

# Reduce exposure of internal systems in alerts
# Replace external network definition with internal-safe view for insider threat detection
emerging-scan "$EXTERNAL_NET" "any"
emerging-sql "$EXTERNAL_NET" "any"
emerging-web_server "$EXTERNAL_NET" "any"
emerging-malware "$EXTERNAL_NET" "any"

# ========================================================
# 🌐 2. PORT CONTEXT HARDENING
# ========================================================

# Force HTTPS inspection instead of HTTP only detection
"HTTP_PORTS" "ANY"
"HTTP_PORTS" "HTTPS_PORTS"

# Expand web inspection coverage
"WEB_PORTS" "80,443,8080,8443"

# ========================================================
# 🧠 3. INTERNAL NETWORK VISIBILITY ENHANCEMENT
# ========================================================

# Apply rules to detect INSIDER THREATS (not just external attacks)
"EXTERNAL_NET" "any"
"$EXTERNAL_NET" "any"
"$HOME_NET" "any"

# ========================================================
# 🔥 4. HIGH-RISK RULE TARGETING (MULTI-SID MODIFICATION)
# ========================================================

# Strengthen detection sensitivity for critical exploit rules
302,429,1821 "$EXTERNAL_NET" "$HOME_NET"
10010 "$HTTP_SERVERS" "$HOME_NET"
10010 "to_client" "from_server"

# ========================================================
# 🛡️ 5. LATERAL MOVEMENT DETECTION HARDENING
# ========================================================

# Expand internal scanning detection scope
emerging-scan "$HOME_NET" "any"
emerging-rpc "$EXTERNAL_NET" "$HOME_NET"
emerging-bruteforce "$EXTERNAL_NET" "$HOME_NET"

# ========================================================
# ⚙️ 6. PROTOCOL BEHAVIOR NORMALIZATION
# ========================================================

# Improve protocol accuracy and reduce blind spots
"TCP_PORTS" "ANY"
"UDP_PORTS" "ANY"

# Ensure full inspection of Windows management services
"MSRPC_PORTS" "135,139,445,593,5985,47001"

############################################################
# END OF MODIFY SID CONFIGURATION
############################################################
```

### `disablesid.conf`

```conf
############################################################
# DISABLE SID CONFIGURATION - SOC TUNING LAYER (PRODUCTION)
############################################################

# --------------------------------------------------------
# 🔹 CORE PRINCIPLE
# This file disables noisy or low-value IDS rules
# Use carefully in production environments
# --------------------------------------------------------

# ========================================================
# 🔕 1. KNOWN HIGH-NOISE RULES (SAFE TO DISABLE)
# ========================================================

# Common false positives / noisy detection rules
1:1034,1:9837,1:1270,1:3390,1:710,1:1249,3:13010

# ========================================================
# 🔕 2. LARGE RANGE NOISE REDUCTION (TUNING BLOCKS)
# ========================================================

# Disable low-value or overly broad detection ranges
1:220-1:3264
3:13010-3:13013

# ========================================================
# 🔕 3. LEGACY / LOW THREAT INTELLIGENCE RULES
# ========================================================

# Old Microsoft exploit patterns with low modern relevance
MS00-\d+,MS01-\d+,MS02-\d+

# Old CVE-based detection rules (low exploitation relevance)
cve:2000-\d+,cve:2001-\d+,cve:2002-\d+

# ========================================================
# 🔕 4. PCRE-BASED NOISE REDUCTION
# ========================================================

# Disable overly broad or noisy regex detections
pcre:(?i)heartbeat
pcre:(?i)test
pcre:(?i)example
pcre:(?i)debug

# Reduce noisy CMS detections in non-public environments
pcre:(?i)wordpress
pcre:(?i)joomla

# ========================================================
# 🔕 5. CATEGORY-BASED DISABLEMENT (FINE-GRAINED CONTROL)
# ========================================================

# Disable low-value scan detection categories (in controlled environments)
emerging-scan
emerging-test
emerging-deleted
etpro-test
etpro-deleted

# Disable legacy shellcode detection if environment is hardened internally
shellcode

# ========================================================
# 🔕 6. HIGH-NOISE WEB ATTACK RULES (OPTIONAL TUNING)
# ========================================================

# Disable noisy web attack signatures in internal-only networks
emerging-web_server
emerging-web_client

# ========================================================
# ⚠️ 7. EXPLICIT NOTES (DO NOT ACTIVATE THIS FILE BLINDLY)
# ========================================================

# This file should be environment-specific:
# - Enable selectively per subnet (LAN vs DMZ vs WAN)
# - Validate with baseline traffic before production deployment
# - Review monthly with SIEM false-positive reports

############################################################
# END OF DISABLE SID CONFIGURATION
############################################################
```
