---
title: GOAD - IP Hostname Matrix
category: Network
environment: GOAD
tags:
  - GOAD
  - Network
  - IP
  - Topology
date_created: 2026-08-23
status: PARTIALLY-VERIFIED
---

# GOAD — IP / Hostname Matrix

> [!CAUTION]
> This matrix contains **documented** IPs, not **verified** IPs.
> Every entry marked VERIFY must be confirmed against the running GOAD deployment before use.
> See [[GOAD - Error Audit and Contradiction Report]] for full details on the conflicts.

> [!WARNING]
> This documentation is for the **authorized GOAD lab environment only**.
> 192.168.56.0/24 is a VirtualBox host-only network. These IPs are not routable on the internet.

---

## Prerequisites

- [ ] GOAD environment is running (`vagrant status` from GOAD directory)
- [ ] VirtualBox host-only adapter is configured for 192.168.56.0/24
- [ ] Kali machine is attached to the host-only adapter

---

## Topology Overview

The GOAD lab runs on `192.168.56.0/24` (VirtualBox host-only adapter).

```
192.168.56.0/24
├── Kali (attacker)          192.168.56.1  ← documented (vmnet7); VERIFY before C2 use
├── KINGSLANDING             192.168.56.10 ← verified 2026-08-23
├── WINTERFELL               192.168.56.11 ← verified 2026-08-23
├── MEEREEN                  192.168.56.12 ← verified 2026-08-23
├── CASTELBLACK              192.168.56.22 ← verified 2026-08-23
└── BRAAVOS                  192.168.56.23 ← verified 2026-08-23
```

Domain Trust Structure (to be verified):
```
sevenkingdoms.local  (root forest)
  └── north.sevenkingdoms.local  (child domain, bidirectional trust)

essos.local  (separate forest, external trust to sevenkingdoms.local)
```

---

## IP / Hostname Matrix

| Host | FQDN | Short Name | Documented IP | Source | Status |
|---|---|---|---|---|---|
| King's Landing | `kingslanding.sevenkingdoms.local` | `kingslanding` | `192.168.56.10` | nmap scan | VERIFIED |
| Winterfell | `winterfell.north.sevenkingdoms.local` | `winterfell` | `192.168.56.11` | nmap scan | VERIFIED |
| Meereen | `meereen.essos.local` | `meereen` | `192.168.56.12` | nmap scan | VERIFIED |
| Castle Black | `castelblack.north.sevenkingdoms.local` | `castelblack` | `192.168.56.22` | nmap scan | VERIFIED |
| Braavos | `braavos.essos.local` | `braavos` | `192.168.56.23` | nmap scan | VERIFIED |
| Kali (attacker) | N/A | kali | `192.168.56.1` | `ip addr` (vmnet7) | VERIFIED |

> [!IMPORTANT]
> The GOAD Assault Playbook note in this vault uses entirely different hostnames and IPs.
> **Do not mix hostnames from different source notes.** The conflict is documented in [[GOAD - Error Audit and Contradiction Report]] (ERROR 01).

---

## Roles (Based on /etc/hosts + Official GOAD Documentation)

| Host | Domain | Expected Role |
|---|---|---|
| KINGSLANDING | `sevenkingdoms.local` | Domain Controller (root forest) |
| WINTERFELL | `north.sevenkingdoms.local` | Domain Controller (child domain) |
| CASTELBLACK | `north.sevenkingdoms.local` | Member server |
| MEEREEN | `essos.local` | Domain Controller (second forest) |
| BRAAVOS | `essos.local` | Member server |

> [!NOTE]
> Role assignments above are based on the official GOAD project topology from Orange Cyberdefense.
> VERIFY these roles against your actual deployment — vagrant configurations may vary.

---

## How to Verify IPs From Kali

```bash
Execution Context: [KALI]
Shell: Bash
Privileges: User
```

### Step 1 — Confirm Kali's own IP on the GOAD network

```bash
ip addr show
# Look for interface with 192.168.56.x address
# Note the interface name (e.g., eth1, eth0, vboxnet0)
```

### Step 2 — Sweep the network

```bash
# Adjust interface name to match your Kali configuration
sudo nmap -sn 192.168.56.0/24
```

Expected result: Live hosts at the IPs listed above.

### Step 3 — Resolve hostnames

```bash
for ip in 192.168.56.10 192.168.56.11 192.168.56.12 192.168.56.22 192.168.56.23; do
    echo "=== $ip ==="
    nslookup $ip 2>/dev/null || echo "No reverse DNS"
done
```

### Step 4 — Record observed results

Fill in the table below with actual observed values:

| IP | Observed Hostname | Observed Role | Verified Date |
|---|---|---|---|
| 192.168.56.10 | kingslanding.sevenkingdoms.local | DC | 2026-08-23 |
| 192.168.56.11 | winterfell.north.sevenkingdoms.local | DC | 2026-08-23 |
| 192.168.56.12 | meereen.essos.local | DC | 2026-08-23 |
| 192.168.56.22 | castelblack.north.sevenkingdoms.local | Member | 2026-08-23 |
| 192.168.56.23 | braavos.essos.local | Member | 2026-08-23 |

---

## Expected Result

After verification, you should be able to ping each host by hostname:

```bash
ping -c 2 kingslanding.sevenkingdoms.local
ping -c 2 winterfell.north.sevenkingdoms.local
ping -c 2 meereen.essos.local
ping -c 2 castelblack.north.sevenkingdoms.local
ping -c 2 braavos.essos.local
```

> [!TIP]
> Successful ping does NOT prove all services are running. Verify specific ports separately.

---

## If It Fails

| Symptom | Possible Cause | Action |
|---|---|---|
| No hosts respond | GOAD VMs not running | `vagrant up` from GOAD directory |
| Wrong IP responds | /etc/hosts conflict | Review [[GOAD - Network Configuration and hosts Procedure]] |
| Hostname resolves but wrong IP | DNS caching | `sudo systemd-resolve --flush-caches` |
| Ping succeeds but SMB fails | Firewall rule | Check Windows Firewall on target |

---

## Related Notes

- [[GOAD - Error Audit and Contradiction Report]]
- [[GOAD - Network Configuration and hosts Procedure]]
- [[GOAD - Environment Setup and Validation]]
