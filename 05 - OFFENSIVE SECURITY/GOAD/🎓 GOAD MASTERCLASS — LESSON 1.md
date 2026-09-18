🔥 **LET’S GO, PROF MODE ACTIVATED.**


# 🎓 GOAD MASTERCLASS — LESSON 1

## Active Directory Reconnaissance

Our first objective is **NOT to exploit anything**.

Our objective is to answer:

> **"What exactly am I looking at?"**

Mayfly starts his GOAD series the same way: identify hosts, domains, domain controllers, DNS, Kerberos, LDAP, SMB and other exposed services. ([Mayfly](https://mayfly277.github.io/posts/GOADv2-pwning_part1/?utm_source=chatgpt.com "GOAD - part 1 - reconnaissance and scan | Mayfly"))

---

# 🧠 The mindset

Imagine the client gives you:

```text
Attacker machine
      │
      │
      ▼
192.168.56.0/24
```

That's all you know.

You don't know:

- domain name
    
- domain controllers
    
- users
    
- computers
    
- services
    
- trusts
    
- certificate authorities
    
- SQL servers
    
- Exchange
    
- administrator accounts
    

Your job is to turn:

```text
192.168.56.0/24
```

into an **AD attack map**.

---

# Step 1 — Confirm your GOAD network

On Kali:

```bash
ip addr
```

Then:

```bash
ip route
```

We want to identify the interface connected to your GOAD network.

You should see something conceptually like:

```text
192.168.56.0/24
```

Then test connectivity:

```bash
ping -c 3 192.168.56.10
```

If your DC isn't `.10`, use the IP of your actual DC.

### What are we testing?

Not exploitation.

We're establishing:

```text
Kali
  │
  └── Can I reach GOAD?
```

---

# Step 2 — Discover the machines

Start with:

```bash
nmap -sn 192.168.56.0/24
```

This is **host discovery**.

We're asking:

> Which IPs are alive?

You may get something like:

```text
192.168.56.10
192.168.56.11
192.168.56.12
192.168.56.21
192.168.56.22
192.168.56.23
```

Don't immediately start exploiting them.

Build a table.

|IP|Hostname|Domain|Likely role|
|---|---|---|---|
|.10|?|?|?|
|.11|?|?|?|
|.12|?|?|?|
|.21|?|?|?|
|.22|?|?|?|
|.23|?|?|?|

This table will become extremely important later.

---

# Step 3 — Full TCP reconnaissance

Now:

```bash
sudo nmap -Pn -p- --min-rate 3000 192.168.56.10
```

Replace `.10` with each discovered host.

Then take the interesting ports and perform service detection:

```bash
sudo nmap -Pn -sC -sV -p 53,80,88,135,139,389,445,464,636,3268,3269,5985,5986 192.168.56.10
```

### Why two scans?

Because we're separating:

```text
Discovery
   ↓
What ports exist?
```

from:

```text
Identification
   ↓
What services are behind those ports?
```

That's professional methodology.

---

# Step 4 — Learn the AD ports

This is where I want you to stop memorizing and start understanding.

### 53 — DNS

```text
DNS
 ↓
Domain names
 ↓
Domain controllers
 ↓
AD infrastructure
```

AD heavily depends on DNS.

---

### 88 — Kerberos

```text
Kerberos
 ↓
Authentication
 ↓
Tickets
 ↓
AD identities
```

Later we'll spend a **whole lesson** on Kerberos.

---

### 389 — LDAP

This is extremely important.

```text
LDAP
 ↓
Active Directory directory
 ↓
Users
Groups
Computers
OUs
Policies
Relationships
```

Think:

> **LDAP = the database interface to the AD directory.**

---

### 445 — SMB

```text
SMB
 ↓
Shares
Files
Remote administration
Windows networking
```

SMB becomes extremely important for lateral movement.

---

### 464 — Kerberos password operations

Related to Kerberos password changes.

---

### 636 — LDAPS

LDAP over TLS.

---

### 3268 / 3269 — Global Catalog

These become particularly important when dealing with **multiple domains in a forest**.

---

### 5985 / 5986 — WinRM

These are Windows Remote Management.

You were literally troubleshooting **5986** in your GOAD lab recently.

Now you know why it's interesting:

```text
5985 → WinRM HTTP
5986 → WinRM HTTPS
```

If we eventually obtain appropriate credentials and permissions, these can become remote-management paths.

---

# Step 5 — Identify the Domain Controller

Here's where things become interesting.

Try:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.sevenkingdoms.local
```

Obviously, replace the domain with the one your lab actually uses.

Mayfly uses this exact conceptual approach to locate the DC through DNS SRV records. ([Mayfly](https://mayfly277.github.io/posts/GOADv2-pwning_part1/?utm_source=chatgpt.com "GOAD - part 1 - reconnaissance and scan | Mayfly"))

Why does this work?

Because Active Directory publishes service records.

You're essentially asking DNS:

> "Which machine provides LDAP for this domain?"

DNS answers:

```text
DC01.domain.local
```

Now you know:

```text
Domain
   ↓
Domain Controller
```

---

# Step 6 — SMB reconnaissance

Now let's inspect SMB.

```bash
nxc smb 192.168.56.10
```

If your version uses the older CrackMapExec syntax, you may see:

```bash
crackmapexec smb 192.168.56.10
```

We're looking for information such as:

```text
OS
Hostname
Domain
Signing
SMB version
```

For example:

```text
Windows Server 2019
DC01
domain.local
SMB signing: True
```

That's already valuable.

---

# 🚨 SMB signing

This deserves attention.

Suppose you discover:

```text
SMB signing: False
```

Don't immediately think:

> "I can hack it."

Think:

> **"This machine may be relevant to NTLM relay attacks."**

Mayfly specifically highlights machines with signing disabled and explains that SMB signing is important for preventing certain NTLM relay attacks. ([Mayfly](https://mayfly277.github.io/posts/GOADv2-pwning_part1/?utm_source=chatgpt.com "GOAD - part 1 - reconnaissance and scan | Mayfly"))

So your reconnaissance notes should record:

```text
HOST
 ├── SMB
 ├── Signing: TRUE/FALSE
 └── Potential relay relevance
```

---

# Step 7 — DNS reconnaissance

Try:

```bash
dig @192.168.56.10 -t SRV _ldap._tcp.dc._msdcs.domain.local
```

Then:

```bash
dig @192.168.56.10 domain.local
```

And:

```bash
dig @192.168.56.10 -t NS domain.local
```

Again, substitute your actual domain.

We're learning:

```text
DNS
 │
 ├── Name servers
 ├── Domain controllers
 ├── Host records
 └── AD service records
```

---

# Step 8 — Build your first attack map

By now, you should have something like:

```text
                    GOAD
                     │
              192.168.56.0/24
                     │
        ┌────────────┼────────────┐
        │            │            │
       DC01         SRV01        SRV02
        │
        ├── DNS :53
        ├── Kerberos :88
        ├── LDAP :389
        ├── SMB :445
        ├── LDAPS :636
        ├── GC :3268
        └── WinRM :5985/5986
```

But we're not finished.

---

# 🧠 The professional question

Don't ask:

> "What ports are open?"

Ask:

> **"What does each service tell me about the architecture?"**

For example:

```text
88 open
   ↓
Kerberos exists
   ↓
Likely AD environment
   ↓
Find domain
   ↓
Find DC
   ↓
Enumerate users
```

Or:

```text
389 open
   ↓
LDAP
   ↓
AD directory
   ↓
Enumerate objects
   ↓
Users / groups / computers
```

Or:

```text
445 open
   ↓
SMB
   ↓
Shares / Windows networking
   ↓
Potential authentication
   ↓
Potential lateral movement
```

That's how you need to think.

---

# 🧪 YOUR FIRST LAB ASSIGNMENT

Don't run anything destructive.

Run these **five things**:

```bash
ip addr
```

```bash
ip route
```

```bash
nmap -sn 192.168.56.0/24
```

Then identify your DC IP and run:

```bash
sudo nmap -Pn -sC -sV -p 53,80,88,135,139,389,445,464,636,3268,3269,5985,5986 <DC_IP>
```

Then:

```bash
nxc smb <DC_IP>
```

---

## 🎯 Send me the output

**Don't clean it up.**

Paste the raw output here.

I'll act as your instructor and we'll go through it **line by line**:

```text
IP
 ↓
Hostname
 ↓
Domain
 ↓
DC
 ↓
Services
 ↓
Security controls
 ↓
Potential attack surface
```

Then we'll move into **Lesson 2: Finding Users & Anonymous Enumeration**.

And from that point onward, I'll make you explain **why** each technique works before we move to the next one.

That's how we're going to get you from _"I can run NetExec"_ to **"I understand Active Directory attack paths."** 🔥

Excellent. 🔥 **This is a very good first reconnaissance pass.** You have already uncovered the core GOAD architecture.

Now I'm going to teach you how to **read this like an AD pentester**, rather than just looking at open ports.

# Lesson 1 — Recon Analysis

Your network has **6 GOAD Windows hosts + Kali**:

|IP|Hostname|Domain|Initial assessment|
|---|---|---|---|
|`192.168.56.10`|`KINGSLANDING`|`sevenkingdoms.local`|**Domain Controller**|
|`192.168.56.11`|`WINTERFELL`|`north.sevenkingdoms.local`|**Domain Controller**|
|`192.168.56.12`|`ESSOS`|`essos.local`|**Domain Controller**|
|`192.168.56.21`|Unknown|Unknown|Windows member host|
|`192.168.56.22`|`CASTELBLACK`|`north.sevenkingdoms.local`|Windows server + MSSQL|
|`192.168.56.23`|`BRAAVOS`|`essos.local`|Windows server + MSSQL|

And:

```text
192.168.56.1
    ↓
KALI
```

---

# 🧠 Stop here: you just discovered something important

Look at these three:

```text
192.168.56.10
sevenkingdoms.local

192.168.56.11
north.sevenkingdoms.local

192.168.56.12
essos.local
```

This isn't simply:

```text
6 Windows machines
```

You've discovered an **Active Directory forest/domain structure**.

Conceptually:

```text
                    AD FOREST
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
 sevenkingdoms.local  essos.local  ?
       │
       │
       ▼
 north.sevenkingdoms.local
```

More precisely, `north.sevenkingdoms.local` appears to be a **child domain** of `sevenkingdoms.local`.

That's a major finding.

---

# 1. Why `.10` is almost certainly a DC

Your `.10` scan shows:

```text
53    DNS
88    Kerberos
389   LDAP
445   SMB
464   Kerberos password
636   LDAPS
3268  Global Catalog
3269  Global Catalog SSL
9389  AD Web Services
```

That's basically screaming:

> **"I am an Active Directory Domain Controller."**

Especially the combination:

```text
88 + 389 + 3268 + 3269 + 9389
```

### Let's understand each one.

---

## 53 — DNS

```text
KALI
  │
  │ DNS query
  ▼
DC
```

AD depends heavily on DNS.

It isn't just translating:

```text
google.com → IP
```

Active Directory uses DNS records to locate services.

For example:

```text
_ldap._tcp.dc._msdcs.domain.local
```

can identify domain controllers.

---

# 2. 88 — Kerberos

This is one of the most important ports in your entire lab.

```text
88/tcp
```

means Kerberos is available.

Kerberos is the primary authentication protocol used by modern Active Directory.

Eventually you're going to learn:

```text
User
 ↓
Kerberos
 ↓
TGT
 ↓
TGS
 ↓
Service
```

And attacks such as:

```text
AS-REP Roasting
Kerberoasting
Pass-the-Ticket
Delegation abuse
```

will make much more sense once you understand this diagram.

---

# 3. 389 — LDAP

Your scan literally tells us:

```text
Microsoft Windows Active Directory LDAP
Domain: sevenkingdoms.local
```

🔥 This is extremely valuable.

LDAP is an interface into the AD directory.

Think:

```text
Active Directory
       │
       ├── Users
       ├── Groups
       ├── Computers
       ├── OUs
       ├── SPNs
       ├── GPO relationships
       └── Permissions
```

LDAP lets clients query that directory.

So eventually we're going to ask:

> "What can an unauthenticated user learn?"

and then:

> "What can an authenticated low-privilege user learn?"

Those are very different questions.

---

# 4. 3268 — Global Catalog

This is an excellent discovery.

```text
3268 Global Catalog LDAP
3269 Global Catalog LDAP over SSL
```

Why?

Because your environment has multiple domains.

The Global Catalog provides a forest-wide searchable directory of selected attributes.

So instead of thinking:

```text
sevenkingdoms.local
```

you should start thinking:

```text
FOREST
   │
   ├── sevenkingdoms.local
   │
   ├── north.sevenkingdoms.local
   │
   └── essos.local
```

And now **cross-domain enumeration** becomes important.

---

# 5. Your certificate gave us another clue

Look at:

```text
Subject:
commonName=kingslanding.sevenkingdoms.local
```

And:

```text
NetBIOS name:
KINGSLANDING
```

So we can establish:

```text
192.168.56.10
        │
        ├── Hostname: KINGSLANDING
        ├── FQDN: kingslanding.sevenkingdoms.local
        └── Domain: sevenkingdoms.local
```

This is exactly why service enumeration matters.

Nmap isn't simply saying:

> "Port 389 is open."

It's giving us **identity information**.

---

# 6. Your `.11` is fascinating

You haven't run service detection against `.11` yet, but look at its ports:

```text
53
88
135
139
389
445
464
593
636
3268
3269
3389
5985
5986
9389
47001
```

That is almost identical to `.10`.

So:

```text
192.168.56.11
WINTERFELL
north.sevenkingdoms.local
```

is almost certainly another **Domain Controller**.

And because:

```text
north.sevenkingdoms.local
```

is a child domain, we now have:

```text
sevenkingdoms.local
        │
        └── north.sevenkingdoms.local
                 │
                 └── WINTERFELL DC
```

---

# 7. `.12` is another DC

Same story:

```text
53
88
389
445
464
636
3268
3269
9389
```

Therefore:

```text
192.168.56.12
ESSOS
essos.local
```

is almost certainly an independent domain in the GOAD forest.

So your initial architecture is shaping up as:

```text
                       FOREST
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
     sevenkingdoms.local          essos.local
             │                       │
             ▼                       ▼
 north.sevenkingdoms.local       ESSOS DC
             │
             ▼
        WINTERFELL DC
```

And:

```text
KINGSLANDING
     │
     └── sevenkingdoms.local DC
```

---

# 8. Now look at `.22`

This one is very different:

```text
80
135
139
445
1433
3389
5985
5986
```

The standout is:

```text
1433/tcp
```

That's:

> **Microsoft SQL Server**

And the hostname is:

```text
CASTELBLACK
```

So:

```text
CASTELBLACK
     │
     ├── IIS
     ├── SMB
     ├── MSSQL :1433
     ├── RDP
     └── WinRM
```

This is going to become **very important later**.

Remember our curriculum:

```text
Recon
 ↓
Credentials
 ↓
MSSQL
 ↓
Privilege escalation
 ↓
Lateral movement
```

You have just found one of our future attack surfaces.

**But we are NOT attacking SQL yet.**

We're still mapping the environment.

---

# 9. `.23` is the same story

```text
BRAAVOS
essos.local
```

has:

```text
80
445
1433
3389
5985
5986
```

Again:

```text
BRAAVOS
   │
   ├── IIS
   ├── SMB
   ├── MSSQL
   ├── RDP
   └── WinRM
```

So your MSSQL attack surface currently looks like:

```text
CASTELBLACK ── MSSQL :1433
BRAAVOS     ── MSSQL :1433
```

---

# 10. `.21` is our mystery machine

This is actually useful.

It only shows:

```text
135
445
3389
5985
5986
```

And:

```text
65530 filtered
```

Unlike the DCs, it doesn't expose:

```text
53
88
389
636
3268
3269
9389
```

Therefore it is **not presenting like a domain controller**.

It looks more like a Windows member machine/server.

But we don't know:

```text
hostname
domain
role
```

yet.

That's our next little investigation.

---

# 11. Your first professional attack-surface map

Let's document what we know.

```text
                    GOAD
                      │
              192.168.56.0/24
                      │
       ┌──────────────┼───────────────┐
       │              │               │
       ▼              ▼               ▼
   KINGSLANDING    WINTERFELL       ESSOS
    .10             .11             .12
       │              │               │
       ▼              ▼               ▼
 sevenkingdoms     north.          essos
    .local       sevenkingdoms     .local
                     .local
       │
       │
       ├──────────────┐
       ▼              ▼
 CASTELBLACK       ? .21
    .22
       │
       └── MSSQL
       
                         BRAAVOS
                           .23
                            │
                            └── MSSQL
```

Already we're seeing **domains + DCs + member servers + SQL**.

That's a proper initial reconnaissance picture.

---

# 12. One important observation: SMB signing

Your `.10` result says:

```text
smb2-security-mode:
3.1.1:
Message signing enabled and required
```

That's good defensive configuration.

It means we shouldn't simply assume:

```text
SMB open
   ↓
NTLM relay
```

No.

We need to enumerate the **actual relay prerequisites**.

This is an important professional habit:

> **An open port is not automatically a vulnerability.**

We'll investigate signing on the other machines later.

---

# 13. One more interesting finding: HTTP TRACE

Nmap reported:

```text
Potentially risky methods:
TRACE
```

Don't get excited yet.

TRACE being enabled is **not automatically a critical vulnerability**.

This is exactly the kind of thing you'll learn to distinguish:

```text
Finding
   ↓
Is it actually exploitable?
   ↓
What's the security impact?
   ↓
Can it contribute to an attack chain?
```

A pentester shouldn't turn every scanner warning into a "critical vulnerability."

---

# 14. Your next assignment

Now we're going to identify the unknowns properly.

### A. Identify `.21`

Run:

```bash
nxc smb 192.168.56.21
```

Then:

```bash
nmap -Pn -sC -sV -p 135,445,3389,5985,5986 192.168.56.21
```

---

### B. Fingerprint the other two servers

Run:

```bash
nxc smb 192.168.56.22
```

```bash
nxc smb 192.168.56.23
```

This should give us useful Windows/domain information.

---

### C. Get service details from the two SQL servers

Run:

```bash
sudo nmap -Pn -sC -sV -p 80,135,139,445,1433,3389,5985,5986 192.168.56.22
```

and:

```bash
sudo nmap -Pn -sC -sV -p 80,135,139,445,1433,3389,5985,5986 192.168.56.23
```

---

### D. Confirm the AD domains through DNS

From Kali:

```bash
dig @192.168.56.10 -t SRV _ldap._tcp.dc._msdcs.sevenkingdoms.local
```

Then:

```bash
dig @192.168.56.11 -t SRV _ldap._tcp.dc._msdcs.north.sevenkingdoms.local
```

And:

```bash
dig @192.168.56.12 -t SRV _ldap._tcp.dc._msdcs.essos.local
```

---

# 🎯 But here's your first test

Before you run anything else, **tell me what you think `.10`, `.11`, `.12`, `.21`, `.22`, and `.23` are and why.**

Don't worry about being wrong.

I want you to reason from the evidence:

```text
.10 → ?
Why?

.11 → ?
Why?

.12 → ?
Why?

.21 → ?
Why?

.22 → ?
Why?

.23 → ?
Why?
```

Then I'll grade your reasoning like a real AD pentesting exercise, correct anything that's wrong, and we'll continue to **Lesson 2: Domain & User Enumeration**. 🔥

🔥 **Excellent work. This is exactly what I wanted to see.**

You didn't just discover hosts—you've now established the **actual AD topology and several attack-surface characteristics**.

## 🎓 Instructor grading

I'd give this reconnaissance pass **9/10**.

The one thing we haven't completely established yet is the exact forest/trust relationship between `sevenkingdoms.local` and `essos.local`. We have strong evidence of the domains, but we should **enumerate that relationship rather than assume it**.

Let's analyze your findings.

---

# 1. Your six machines

We can now confidently classify them:

|IP|Host|Domain|Role|
|---|---|---|---|
|`.10`|KINGSLANDING|`sevenkingdoms.local`|**Domain Controller**|
|`.11`|WINTERFELL|`north.sevenkingdoms.local`|**Domain Controller**|
|`.12`|MEEREEN|`essos.local`|**Domain Controller**|
|`.21`|THE-EYRIE|`sevenkingdoms.local`|**Member server/workstation**|
|`.22`|CASTELBLACK|`north.sevenkingdoms.local`|**Member server + MSSQL**|
|`.23`|BRAAVOS|`essos.local`|**Member server + MSSQL**|

The `.21` result is particularly useful:

```text
THE-EYRIE
domain: sevenkingdoms.local
signing: False
```

Your RDP enumeration independently confirms:

```text
THE-EYRIE
DNS domain: sevenkingdoms.local
DNS computer: the-eyrie.sevenkingdoms.local
```

So our original mystery machine is solved.

---

# 2. The biggest discovery: SMB signing

Look at THE-EYRIE:

```text
signing:False
```

and Nmap:

```text
Message signing enabled but not required
```

CASTELBLACK is the same:

```text
signing:False
```

and:

```text
Message signing enabled but not required
```

BRAAVOS goes even further:

```text
SMBv1:True
```

and Nmap reports:

```text
message_signing: disabled
```

### 🚨 This is extremely interesting for our lab.

But I want you to learn the professional distinction:

**"SMB signing not required" ≠ "machine is automatically compromised."**

Instead:

```text
SMB signing not required
          ↓
Potential NTLM relay exposure
          ↓
Need to determine:
    ├── Can authentication be coerced?
    ├── Where can it be relayed?
    ├── What privileges does the relayed identity have?
    └── Are other protections enabled?
```

This is exactly the type of finding we'll investigate later.

**Do not relay anything yet.**

We're still in reconnaissance.

---

# 3. BRAAVOS is our first obvious hardening finding

This is particularly interesting:

```text
Windows Server 2016
SMBv1: True
SMB signing disabled
```

From a defensive assessment perspective, that's already worth documenting.

You have:

```text
BRAAVOS
 │
 ├── SMBv1 enabled
 ├── SMB signing disabled
 ├── MSSQL 2019
 ├── IIS
 ├── RDP
 └── WinRM
```

That's a very interesting machine.

---

# 4. CASTELBLACK gives us another major clue

Your SQL enumeration says:

```text
Microsoft SQL Server 2019 RTM
15.00.2000.00
Post-SP patches applied: false
```

So this isn't merely:

```text
CASTELBLACK → port 1433
```

We now know:

```text
CASTELBLACK
      │
      └── SQL Server 2019 RTM
```

And more importantly, SQL Server identifies itself as:

```text
Target_Name: NORTH
NetBIOS_Domain_Name: NORTH
DNS_Domain_Name: north.sevenkingdoms.local
```

That's excellent enumeration.

We have independently confirmed:

```text
CASTELBLACK
     ↓
NORTH
     ↓
north.sevenkingdoms.local
```

---

# 5. BRAAVOS confirms the ESSOS domain

Same process:

```text
Target_Name: ESSOS
NetBIOS_Domain_Name: ESSOS
NetBIOS_Computer_Name: BRAAVOS
DNS_Domain_Name: essos.local
```

And SMB independently gives:

```text
Forest name: essos.local
FQDN: braavos.essos.local
```

So we're not relying on one scanner result.

We're **correlating evidence**.

That's professional recon.

---

# 6. Now look at your DNS results

This is the part I especially want you to understand.

You asked:

```text
_ldap._tcp.dc._msdcs.sevenkingdoms.local
```

and DNS answered:

```text
389
kingslanding.sevenkingdoms.local
192.168.56.10
```

That's beautiful.

You just proved:

```text
sevenkingdoms.local
       ↓
LDAP DC service
       ↓
KINGSLANDING
       ↓
192.168.56.10
```

---

# 7. You did the same thing for NORTH

Your query:

```text
_ldap._tcp.dc._msdcs.north.sevenkingdoms.local
```

returned:

```text
winterfell.north.sevenkingdoms.local
192.168.56.11
```

Therefore:

```text
north.sevenkingdoms.local
          ↓
       WINTERFELL
          ↓
     192.168.56.11
```

And notice the hierarchy:

```text
sevenkingdoms.local
        │
        └── north.sevenkingdoms.local
```

That is why I called `north` a child domain earlier.

---

# 8. And ESSOS

You queried:

```text
_ldap._tcp.dc._msdcs.essos.local
```

and got:

```text
meereen.essos.local
192.168.56.12
```

So:

```text
essos.local
    ↓
MEEREEN
    ↓
192.168.56.12
```

---

# 9. Our current AD map

Now let's build the map properly.

```text
                         GOAD
                           │
                  192.168.56.0/24
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
 sevenkingdoms.local   north.sevenkingdoms   essos.local
        │                  │                  │
        │                  │                  │
        ▼                  ▼                  ▼
 KINGSLANDING          WINTERFELL           MEEREEN
 192.168.56.10        192.168.56.11       192.168.56.12
     DC                    DC                  DC
        │                  │                  │
        ▼                  ▼                  ▼
   THE-EYRIE          CASTELBLACK           BRAAVOS
 192.168.56.21        192.168.56.22       192.168.56.23
   Member               Member              Member
                         │                    │
                         ▼                    ▼
                       MSSQL                MSSQL
```

This is now a **real attack surface map**, not a guess.

---

# 10. But there's a subtle thing you should notice

Look at:

```text
sevenkingdoms.local
```

and:

```text
essos.local
```

They don't have the same DNS suffix.

Compare:

```text
north.sevenkingdoms.local
```

versus:

```text
essos.local
```

So don't automatically draw:

```text
sevenkingdoms
     │
     └── essos
```

We don't yet have enough evidence.

We need to determine whether:

```text
sevenkingdoms.local
        │
        │ trust?
        ▼
essos.local
```

exists.

**That's our next AD concept: trusts.**

But we're not jumping there yet.

---

# 11. There's another important clue: time

Your machines are showing UTC timestamps, and the times are close.

For example, your DC scan showed:

```text
Kerberos server time:
2026-08-28 00:07:47Z
```

And the system time from Nmap is also very close.

Kerberos is extremely sensitive to clock skew.

Remember this:

```text
Kerberos
   ↓
Tickets contain timestamps
   ↓
Client/server clocks need to be synchronized
```

If your Kali clock becomes significantly different from the DC:

```text
Kali ─────────────── DC
  ✕ clock mismatch
        ↓
Kerberos problems
```

This will save you a LOT of frustration later.

---

# 12. One thing I want you to correct in your mental model

Earlier we said:

```text
DC
 ↓
AD database
```

That's correct conceptually.

But don't think:

> "LDAP is the database."

Instead:

```text
Active Directory
      │
      ├── Directory database
      │
      └── LDAP interface
              ↓
       Query/modify directory
```

That's a much better mental model.

---

# 🧪 LESSON 1 EXAM — Part 2

You've passed the first reconnaissance exercise.

Now we're going to move from:

> **"What machines exist?"**

to:

> **"What does Active Directory reveal to an unauthenticated attacker?"**

Before we enumerate users, however, I want you to perform **three controlled reconnaissance checks**.

### 1️⃣ Enumerate SMB information across all Windows hosts

Run:

```bash
nxc smb 192.168.56.10,192.168.56.11,192.168.56.12,192.168.56.21,192.168.56.22,192.168.56.23
```

### 2️⃣ Check SMB signing across all six

Run:

```bash
nxc smb 192.168.56.10,192.168.56.11,192.168.56.12,192.168.56.21,192.168.56.22,192.168.56.23
```

It's the same command because NetExec reports signing in the SMB banner.

I want you to create this table yourself:

|Host|IP|Domain|Signing|SMBv1|
|---|---|---|---|---|
|KINGSLANDING|.10|?|?|?|
|WINTERFELL|.11|?|?|?|
|MEEREEN|.12|?|?|?|
|THE-EYRIE|.21|?|?|?|
|CASTELBLACK|.22|?|?|?|
|BRAAVOS|.23|?|?|?|

### 3️⃣ Then test anonymous LDAP discovery

We'll start carefully with:

```bash
nxc ldap 192.168.56.10
```

and:

```bash
nxc ldap 192.168.56.11
```

and:

```bash
nxc ldap 192.168.56.12
```

**Don't start password attacks. Don't spray credentials. Don't exploit anything yet.**

We're learning enumeration first.

---

## 🎯 Your next checkpoint

Send me the output of those commands.

Then I'll teach you:

# **LESSON 2 — LDAP, Anonymous Enumeration & Users**

And this is where the lab starts becoming really interesting:

```text
Unauthenticated attacker
        ↓
      LDAP
        ↓
   Domain information
        ↓
      Users
        ↓
     Groups
        ↓
      SPNs
        ↓
Potential credential attack paths
```

Once you understand **why** that information is exposed, Kerberoasting, AS-REP roasting, BloodHound, ACL abuse, delegation and the rest of Mayfly's material will start connecting together instead of feeling like 20 unrelated techniques.

🔥 **Perfect. Now we're getting into the real AD enumeration.**

Your results give us two important lessons.

## 1. The `nxc smb` command produced nothing

You ran:

```bash
nxc smb 192.168.56.10,192.168.56.11,...
```

and got no output.

That's not necessarily a problem with the lab. NetExec's target parsing/version behavior can make comma-separated targets behave differently than expected.

Let's do it explicitly:

```bash
for ip in 192.168.56.10 192.168.56.11 192.168.56.12 192.168.56.21 192.168.56.22 192.168.56.23; do
    echo "===== $ip ====="
    nxc smb "$ip"
done
```

This is better for learning too, because we'll see each host separately.

---

# 2. Your LDAP results are VERY interesting

You got:

```text
KINGSLANDING
domain: sevenkingdoms.local
signing: None
channel binding: Never
```

Then:

```text
WINTERFELL
domain: north.sevenkingdoms.local
signing: None
channel binding: Never
```

And:

```text
MEEREEN
domain: essos.local
signing: None
channel binding: Never
```

So our domain/DC map is now confirmed:

```text
                    GOAD FOREST
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
 sevenkingdoms.local  north...       essos.local
          │          .seven...            │
          ▼              ▼              ▼
    KINGSLANDING      WINTERFELL       MEEREEN
      192.168.56.10   192.168.56.11   192.168.56.12
```

---

# 🧠 Now understand these two fields

This is where we're going deeper than simply following Mayfly commands.

## `signing: None`

NetExec is telling you that LDAP signing isn't being advertised/enforced in this connection.

That does **not** mean:

> "LDAP is vulnerable."

It means:

> **"We need to investigate the LDAP security configuration further."**

---

## `channel binding: Never`

This is another security control.

Channel binding helps protect certain authentication exchanges by binding the authentication to the underlying secure channel.

So your reconnaissance notes should now contain:

```text
KINGSLANDING
 ├── LDAP signing: None
 └── Channel binding: Never

WINTERFELL
 ├── LDAP signing: None
 └── Channel binding: Never

MEEREEN
 ├── LDAP signing: None
 └── Channel binding: Never
```

This becomes particularly relevant when we study **NTLM relay** later.

Again:

**Don't attack it yet.**

We're learning how to recognize the prerequisite conditions first.

---

# 🎯 Now we're entering Lesson 2

Our question changes.

Previously:

> **What machines exist?**

Now:

> **What can an unauthenticated attacker learn about the directory?**

That's a fundamental distinction.

---

# Step 1 — Test anonymous LDAP access

Try:

```bash
ldapsearch -x \
-H ldap://192.168.56.10 \
-s base \
-b "" \
"(objectClass=*)"
```

### Understand the command

```text
ldapsearch
   │
   ├── -x
   │    Simple authentication
   │
   ├── -H
   │    LDAP server
   │
   ├── -s base
   │    Query only the root object
   │
   ├── -b ""
   │    Root of the LDAP directory
   │
   └── filter
        What we're searching for
```

We're deliberately starting with a **small, low-impact query**.

---

# Step 2 — Find the naming context

If the query works, look for:

```text
defaultNamingContext
```

You may see something like:

```text
defaultNamingContext: DC=sevenkingdoms,DC=local
```

This is important.

Because DNS says:

```text
sevenkingdoms.local
```

while LDAP represents the same domain as:

```text
DC=sevenkingdoms,DC=local
```

### Mental model

```text
DNS name

sevenkingdoms.local
       │
       ▼
LDAP naming context

DC=sevenkingdoms,DC=local
```

That translation is something you need to become completely comfortable with.

---

# Step 3 — Query the domain root

Once we know the naming context:

```bash
ldapsearch -x \
-H ldap://192.168.56.10 \
-b "DC=sevenkingdoms,DC=local" \
-s base \
"(objectClass=*)"
```

We're asking:

> "Tell me about the root object of this domain."

---

# Step 4 — Don't dump the entire directory yet

I **don't** want you immediately running:

```bash
ldapsearch ... "(objectClass=*)"
```

across the entire directory and dumping thousands of objects.

Why?

Because we're learning methodology.

A professional assessment generally progresses:

```text
Discovery
   ↓
Identify directory
   ↓
Identify naming context
   ↓
Test anonymous access
   ↓
Determine accessible attributes
   ↓
Enumerate specific object classes
```

Not:

```text
Run giant command
↓
Hope something interesting appears
```

---

# 🧪 Your next exercise

Run these **three commands** against KINGSLANDING:

### ①

```bash
ldapsearch -x -H ldap://192.168.56.10 -s base -b "" "(objectClass=*)"
```

### ②

If you see the naming context, use it:

```bash
ldapsearch -x -H ldap://192.168.56.10 \
-b "DC=sevenkingdoms,DC=local" \
-s base "(objectClass=*)"
```

### ③

And run the same root-DSE query against the other DCs:

```bash
ldapsearch -x -H ldap://192.168.56.11 -s base -b "" "(objectClass=*)"
```

```bash
ldapsearch -x -H ldap://192.168.56.12 -s base -b "" "(objectClass=*)"
```

---

## ⚠️ One thing to watch for

You may get:

```text
result: 0 Success
```

but that doesn't necessarily mean:

> "Anonymous enumeration is wide open."

There is a difference between:

```text
Anonymous connection succeeds
```

and:

```text
Anonymous directory enumeration succeeds
```

That's an important pentesting concept.

---

# 🧠 Your checkpoint

Send me the output.

I'll teach you how to identify:

```text
RootDSE
   │
   ├── defaultNamingContext
   ├── rootDomainNamingContext
   ├── configurationNamingContext
   ├── schemaNamingContext
   ├── dnsHostName
   └── supported capabilities
```

Then we'll move into:

# **Lesson 2B — Enumerating Users, Groups & Computers**

And that's where you'll start seeing the AD attack graph emerge:

```text
                    DOMAIN
                      │
          ┌───────────┼───────────┐
          │           │           │
        Users       Groups     Computers
          │           │           │
          └───────────┼───────────┘
                      │
                 Relationships
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
           SPNs     ACLs    Delegation
             │
             ▼
        Attack paths
```

**You're doing this exactly right.** We aren't rushing to exploitation—we're building the AD mental model first.