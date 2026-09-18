# Playbook: Unbound Local DNSSEC Resolver on Kali Linux

## Overview
This runbook details how to configure **Unbound** as an independent, local recursive DNS resolver on a Kali Linux workstation. 

### Why Use This Architecture?
- **Privacy:** Queries go directly to the root authoritative servers, bypassing ISP or public DNS logging (e.g., 1.1.1.1, 8.8.8.8).
- **Stealth:** QNAME minimization ensures only the minimum required portion of a domain is sent to upstream servers.
- **Security:** Local DNSSEC validation mathematically prevents DNS spoofing and cache poisoning.
- **Autonomy:** Your Kali machine's DNS will not fail if a local Pi-hole goes down, nor will it be affected by restrictive lab network DNS hijacking.

---

## 1. Installation & Preparation

Install the required packages:
```bash
sudo apt update
sudo apt install unbound dnsutils
```

Ensure the root trust anchor (used for DNSSEC) exists and has the correct permissions:
```bash
sudo unbound-anchor -a /var/lib/unbound/root.key
sudo chown unbound:unbound /var/lib/unbound/root.key
```

*(Optional)* If `systemd-resolved` is currently binding to port 53, you must disable its stub listener:
```bash
sudo sed -i 's/#DNSStubListener=yes/DNSStubListener=no/' /etc/systemd/resolved.conf
sudo systemctl restart systemd-resolved
```

---

## 2. Unbound Configuration

Create or modify your local configuration file (e.g., `/etc/unbound/unbound.conf.d/kali-local.conf`):

```yaml
server:
    verbosity: 1

    interface: 127.0.0.1
    port: 53

    # IPv4/TCP/UDP Configuration
    do-ip4: yes
    do-tcp: yes
    do-udp: yes
    
    # Disable IPv6 if the workstation does not have routable IPv6. 
    # Leaving this enabled without IPv6 routing causes severe resolution timeouts.
    do-ip6: no

    num-threads: 2
    cache-min-ttl: 300
    cache-max-ttl: 86400

    # Privacy Options
    hide-identity: yes
    hide-version: yes
    qname-minimisation: yes

    # Security & Hardening
    harden-glue: yes
    harden-dnssec-stripped: yes
    
    # CRITICAL: harden-referral-path must be 'no' for complex Anycast domains 
    # like Cloudflare to prevent upstream query quota exhaustion (SERVFAIL loops).
    harden-referral-path: no

    # Performance
    prefetch: yes
    rrset-roundrobin: yes

    # Access Control
    access-control: 127.0.0.0/8 allow
```

Apply the configuration:
```bash
sudo systemctl enable unbound
sudo systemctl restart unbound
```

---

## 3. NetworkManager & Resolv.conf Configuration

To ensure DHCP does not overwrite your DNS settings when joining new networks, configure NetworkManager to ignore auto-DNS and force `127.0.0.1`.

1. Open your NetworkManager connection profile or set it globally. To set it for a specific connection (e.g., `eth0` or `Wired connection 1`):
   ```bash
   sudo nmcli connection modify "Wired connection 1" ipv4.dns "127.0.0.1"
   sudo nmcli connection modify "Wired connection 1" ipv4.ignore-auto-dns yes
   sudo nmcli connection up "Wired connection 1"
   ```

2. Verify that `/etc/resolv.conf` contains only:
   ```text
   nameserver 127.0.0.1
   ```

---

## 4. Verification & Testing

Verify that DNSSEC and standard resolution are working by querying a signed domain:

```bash
dig cloudflare.com @127.0.0.1 +dnssec
```

**Expected Output Indicators:**
- `status: NOERROR`
- `flags: qr rd ra ad` (The **`ad`** flag means "Authentic Data", proving DNSSEC validation passed).

Verify the root trust anchor is working:
```bash
dig . DNSKEY @127.0.0.1 +dnssec
```

---

## 5. Troubleshooting & Diagnostics

If queries begin returning `SERVFAIL`:

1. **Check Live Statistics and Cache:**
   ```bash
   sudo unbound-control stats_noreset
   sudo unbound-control lookup <failing_domain.com>
   ```

2. **Inspect Unbound Logs:**
   ```bash
   sudo journalctl -u unbound --since "10 minutes ago" --no-pager
   ```

3. **Known Issue: "Maximum global quota on number of upstream queries"**
   If logs show `request has exceeded the maximum global quota on number of upstream queries`, this indicates a query loop. Ensure that `harden-referral-path: no` is set, and flush the cache:
   ```bash
   sudo unbound-control flush_infra all
   sudo unbound-control flush_zone .
   ```

4. **Test External Connectivity Independently:**
   Bypass Unbound to ensure standard internet connectivity is active:
   ```bash
   dig cloudflare.com @1.1.1.1
   ```
