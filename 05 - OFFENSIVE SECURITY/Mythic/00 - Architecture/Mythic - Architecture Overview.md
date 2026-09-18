---
title: Mythic - Architecture Overview
category: Architecture
environment: GOAD
tags:
  - Mythic
  - C2
  - Architecture
  - Apollo
date_created: 2026-08-23
status: REFERENCE
---

# Mythic — Architecture Overview

> [!NOTE]
> This note is a reference document explaining Mythic C2 architecture as it applies to the GOAD lab.
> For installation procedures, see [[Mythic - Installation and Verification]].

---

## What is Mythic?

Mythic is an open-source command-and-control (C2) framework designed for offensive security operations and red team training.

- **Repository:** https://github.com/its-a-feature/Mythic
- **Documentation:** https://docs.mythic-c2.net/
- **License:** BSD 3-Clause

---

## Component Overview

```
Mythic Server (Kali)
├── mythic-cli          — management CLI
├── Docker containers   — one per service
│   ├── mythic_server   — web UI and API (port 7443)
│   ├── mythic_postgres — database
│   ├── mythic_rabbitmq — message queue
│   └── mythic_nginx    — reverse proxy
├── Agents (plugins)
│   ├── Apollo          — C# Windows agent (recommended for GOAD)
│   └── Athena          — multi-platform agent
└── C2 Profiles (plugins)
    ├── http            — plain HTTP callback
    └── https           — encrypted HTTPS callback
```

---

## Agent: Apollo

Apollo is the primary agent used in this GOAD lab.

| Property | Value |
|---|---|
| Language | C# (.NET) |
| Platform | Windows |
| Repository | https://github.com/MythicAgents/Apollo |
| Key capability | `execute-assembly` — loads .NET binaries in-memory |

### Apollo Task Types

| Task | Purpose | Execution Location |
|---|---|---|
| `shell` | Execute cmd.exe command | Windows victim |
| `powershell` | Execute PowerShell | Windows victim |
| `execute-assembly` | Load .NET EXE in memory | Windows victim (never on disk) |
| `upload` | Push file to victim | Kali → victim |
| `download` | Pull file from victim | Victim → Mythic server |
| `socks` | Start SOCKS5 proxy | Windows victim |
| `steal_token` | Steal process token | Windows victim |
| `screenshot` | Capture screen | Windows victim |

> [!IMPORTANT]
> `shell`, `powershell`, `execute-assembly` and all other task commands are issued from the **Mythic UI** to an **active callback** on a Windows victim.
> They are NOT Kali Bash commands. Running `shell hostname` in a Kali terminal will fail.

---

## execute-assembly — Key Concept

`execute-assembly` is the core Mythic technique that enables in-memory .NET binary execution:

1. Mythic loads the .NET binary from the Mythic server's file store
2. Apollo receives the binary over the C2 channel (encrypted)
3. Apollo loads the binary into its own process memory
4. The binary's `Main()` method executes
5. Output is returned through the C2 channel

**Result:** The binary never exists as a file on the victim's disk.

> [!NOTE]
> Some techniques still require files on disk (e.g., GodPotato launching mimikatz.exe as a child process).
> See [[Mythic - Lab Operations - GodPotato and Mimikatz]] for the correct approach.

---

## Network Architecture (GOAD Lab)

```
Kali (192.168.56.x) ── [host-only 192.168.56.0/24] ── GOAD VMs
     │
     └── Mythic Server (port 7443 = UI, port 80/443 = C2)
              │
              └── Apollo agent callbacks from Windows VMs
```

**C2 traffic flow:**
```
Apollo agent (Windows VM) → HTTP GET/POST → Mythic server (Kali) → Operator UI
```

> [!IMPORTANT]
> The `callback_host` in every Apollo payload must be the actual Kali IP on the 192.168.56.0/24 network.
> VERIFY the Kali IP with `ip addr` before generating any payload.

---

## OPSEC Properties

| Property | Value |
|---|---|
| C2 traffic encryption | AES-256 |
| Beacon interval | Configurable (default 10s) |
| Jitter | Configurable (default 30%) |
| Disk artifacts | Minimal (execute-assembly = no disk) |
| Process injection | Available via `psinject`, `assembly_inject` |

---

## Related Notes

- [[Mythic - Installation and Verification]]
- [[Mythic - Payload Generation]]
- [[Mythic - Lab Operations - GodPotato and Mimikatz]]
- [[GOAD - Error Audit and Contradiction Report]]
