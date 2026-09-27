If you mean **set Linux's default IPv4 TTL/hop limit to 65**, the Linux equivalent is:

### Temporary — until reboot

```bash
sudo sysctl -w net.ipv4.ip_default_ttl=65
```

Verify:

```bash
sysctl net.ipv4.ip_default_ttl
```

Expected:

```text
net.ipv4.ip_default_ttl = 65
```

### Permanent

Edit `/etc/sysctl.conf`:

```bash
sudo nano /etc/sysctl.conf
```

Add:

```text
net.ipv4.ip_default_ttl=65
```

Then apply it:

```bash
sudo sysctl -p
```

Verify:

```bash
sysctl net.ipv4.ip_default_ttl
```

### On your Kali Linux

Since you're using Kali, you can simply run:

```bash
sudo sysctl -w net.ipv4.ip_default_ttl=65
```

If you're specifically trying to make **Linux behave like the Windows command you posted**, this is the direct equivalent.