**defensive forensic check of your Kali host**, I’d do this in stages so you preserve evidence before changing anything.

> **Important:** If you suspect an active compromise, avoid rebooting, deleting files, killing processes, or “cleaning” the system until you collect evidence.

## 1. Create a forensic workspace

```bash
mkdir -p ~/forensics/$(date +%Y%m%d_%H%M%S)
export CASE="$HOME/forensics/$(ls -1t ~/forensics | head -1)"
echo "$CASE"
```

Record the basic environment:

```bash
date -u
hostnamectl
uname -a
uptime
whoami
id
pwd
```

Save it:

```bash
{
    date -u
    hostnamectl
    uname -a
    uptime
    whoami
    id
} | tee "$CASE/system-info.txt"
```

---

# 2. Check logged-in users and sessions

```bash
who
w
users
last -a
lastlog
```

More detailed:

```bash
last -Fai
```

Check failed authentication:

```bash
sudo lastb -a
```

If `lastb` isn't available:

```bash
sudo journalctl | grep -Ei 'failed|authentication failure|invalid user|failed password'
```

---

# 3. Check accounts

```bash
cat /etc/passwd
sudo cat /etc/shadow
sudo cat /etc/group
```

Look for UID 0 accounts:

```bash
awk -F: '$3 == 0 {print}' /etc/passwd
```

Look for users with login shells:

```bash
awk -F: '$7 !~ /(nologin|false)$/ {print $1 ":" $7}' /etc/passwd
```

Check sudo configuration:

```bash
sudo cat /etc/sudoers
sudo ls -la /etc/sudoers.d/
sudo grep -RniE 'ALL|NOPASSWD|sudo' /etc/sudoers /etc/sudoers.d/ 2>/dev/null
```

---

# 4. Processes — very important

Current processes:

```bash
ps auxf
```

More detailed:

```bash
ps -ef
```

Processes with network connections:

```bash
sudo ss -tulpn
sudo ss -tpn
```

Check processes running as root:

```bash
ps -U root -u root u
```

Check process tree:

```bash
pstree -alp
```

Look for deleted executables still running:

```bash
sudo lsof +L1
```

Check executable paths:

```bash
sudo ls -l /proc/[0-9]*/exe 2>/dev/null
```

---

# 5. Network forensic checks

Interfaces:

```bash
ip addr
ip link
```

Routing:

```bash
ip route
ip -6 route
```

ARP/neighbour table:

```bash
ip neigh
```

Listening ports:

```bash
sudo ss -lntup
```

All TCP connections:

```bash
sudo ss -antp
```

All UDP sockets:

```bash
sudo ss -anup
```

DNS configuration:

```bash
resolvectl status
cat /etc/resolv.conf
```

Check `/etc/hosts`:

```bash
cat /etc/hosts
```

Check DNS-related modifications:

```bash
ls -la /etc/resolv.conf
```

---

# 6. Firewall

If using nftables:

```bash
sudo nft list ruleset
```

iptables:

```bash
sudo iptables -L -n -v
sudo iptables -S
```

IPv6:

```bash
sudo ip6tables -L -n -v
```

UFW:

```bash
sudo ufw status verbose
```

Check firewall services:

```bash
systemctl --type=service | grep -Ei 'ufw|firewall|nft|iptables'
```

---

# 7. Persistence / startup mechanisms

This is one of the most important areas.

### Systemd

```bash
systemctl list-unit-files --state=enabled
```

Running services:

```bash
systemctl --type=service --state=running
```

Failed services:

```bash
systemctl --failed
```

Recently modified systemd files:

```bash
sudo find /etc/systemd /usr/lib/systemd /lib/systemd \
-type f -mtime -30 -ls 2>/dev/null
```

User services:

```bash
systemctl --user list-unit-files --state=enabled
systemctl --user --type=service --state=running
```

---

# 8. Cron persistence

System cron:

```bash
sudo cat /etc/crontab
```

Cron directories:

```bash
sudo find /etc/cron* -type f -o -type l 2>/dev/null
```

Your cron:

```bash
crontab -l
```

Root cron:

```bash
sudo crontab -l
```

All user crontabs:

```bash
sudo find /var/spool/cron /var/spool/cron/crontabs \
-type f -exec ls -la {} \; 2>/dev/null
```

---

# 9. SSH forensic check

Configuration:

```bash
sudo cat /etc/ssh/sshd_config
```

Effective SSH configuration:

```bash
sudo sshd -T
```

SSH keys:

```bash
find ~/.ssh -maxdepth 2 -type f -ls
```

Check authorized keys:

```bash
cat ~/.ssh/authorized_keys 2>/dev/null
```

Root's keys:

```bash
sudo cat /root/.ssh/authorized_keys 2>/dev/null
```

All authorized keys:

```bash
sudo find /home /root -name authorized_keys -type f \
-exec ls -la {} \; 2>/dev/null
```

SSH logs:

```bash
sudo journalctl -u ssh --no-pager
```

Or:

```bash
sudo grep -Ei 'accepted|failed|invalid|authentication' \
/var/log/auth.log 2>/dev/null
```

---

# 10. Check recently modified files

Last 24 hours:

```bash
sudo find /etc /usr/bin /usr/sbin /bin /sbin /opt /var \
-type f -mtime -1 -ls 2>/dev/null
```

Last 7 days:

```bash
sudo find /etc /usr/bin /usr/sbin /bin /sbin /opt /var \
-type f -mtime -7 -ls 2>/dev/null
```

Files modified recently in your home directory:

```bash
find ~ -type f -mtime -7 -ls
```

Hidden files:

```bash
find ~ -type f -name '.*' -ls
```

---

# 11. Look for suspicious executables

Executable files in your home directory:

```bash
find ~ -type f -executable -ls
```

Executable files in `/tmp`:

```bash
sudo find /tmp /var/tmp /dev/shm \
-type f -executable -ls 2>/dev/null
```

World-writable executable files:

```bash
sudo find / -xdev -type f -perm /o+w -executable \
-ls 2>/dev/null
```

SUID files:

```bash
sudo find / -xdev -type f -perm -4000 -ls 2>/dev/null
```

SGID files:

```bash
sudo find / -xdev -type f -perm -2000 -ls 2>/dev/null
```

---

# 12. Check `/tmp`, `/var/tmp`, `/dev/shm`

```bash
ls -lah /tmp
ls -lah /var/tmp
ls -lah /dev/shm
```

Recursive:

```bash
sudo find /tmp /var/tmp /dev/shm -type f -ls 2>/dev/null
```

Look for recently created files:

```bash
sudo find /tmp /var/tmp /dev/shm \
-type f -mtime -7 -ls 2>/dev/null
```

---

# 13. Kernel modules

Loaded modules:

```bash
lsmod
```

Detailed:

```bash
sudo modinfo <module>
```

Recently loaded module information:

```bash
sudo journalctl -k | grep -Ei 'module|taint'
```

Check kernel messages:

```bash
sudo dmesg -T
```

---

# 14. Package integrity

First update package metadata:

```bash
sudo apt update
```

List installed packages:

```bash
dpkg -l
```

Recently installed packages:

```bash
grep -Ei ' install | upgrade ' /var/log/dpkg.log
```

Current package logs:

```bash
sudo zgrep -Ei ' install | upgrade ' /var/log/dpkg.log*
```

Check package files:

```bash
sudo debsums -c
```

If unavailable:

```bash
sudo apt install debsums
```

Then:

```bash
sudo debsums -c
```

---

# 15. Check shell history

```bash
history
```

Bash history:

```bash
cat ~/.bash_history
```

Zsh history:

```bash
cat ~/.zsh_history
```

Search for suspicious commands:

```bash
grep -Ei 'curl|wget|nc |netcat|ncat|socat|python|perl|bash -i|ssh|chmod|chown|base64|/tmp|/dev/shm' \
~/.bash_history ~/.zsh_history 2>/dev/null
```

Check root history:

```bash
sudo cat /root/.bash_history 2>/dev/null
sudo cat /root/.zsh_history 2>/dev/null
```

---

# 16. Check environment variables

```bash
env
```

Root environment:

```bash
sudo env
```

Look specifically for dangerous PATH modifications:

```bash
echo "$PATH"
sudo sh -c 'echo "$PATH"'
```

Check shell startup files:

```bash
ls -la ~
cat ~/.bashrc
cat ~/.profile
cat ~/.zshrc 2>/dev/null
```

Root:

```bash
sudo cat /root/.bashrc
sudo cat /root/.profile
sudo cat /root/.zshrc 2>/dev/null
```

---

# 17. Check browser persistence

List browser extensions/configuration:

```bash
find ~/.config -maxdepth 4 \
-type d \( -iname '*chrome*' -o -iname '*chromium*' -o -iname '*firefox*' \) \
-print 2>/dev/null
```

Firefox profiles:

```bash
find ~/.mozilla -type f -mtime -30 -ls 2>/dev/null
```

Chromium:

```bash
find ~/.config/chromium -type f -mtime -30 -ls 2>/dev/null
```

---

# 18. Check Docker

Since you use Docker heavily, this is particularly useful.

```bash
docker ps -a
```

Images:

```bash
docker images
```

Networks:

```bash
docker network ls
```

Volumes:

```bash
docker volume ls
```

Running containers:

```bash
docker stats --no-stream
```

Inspect containers:

```bash
docker inspect $(docker ps -aq)
```

Docker events:

```bash
docker events --since 24h --until 0s
```

Docker daemon logs:

```bash
sudo journalctl -u docker --since "7 days ago"
```

Check Docker socket permissions:

```bash
ls -l /var/run/docker.sock
```

Check Docker group membership:

```bash
getent group docker
```

---

# 19. Check VMware / virtualization processes

```bash
ps aux | grep -Ei 'vmware|vbox|qemu|virtualbox'
```

VMware services:

```bash
systemctl --type=service | grep -i vmware
```

Check VMware processes:

```bash
pgrep -a -f 'vmware|vmnet'
```

---

# 20. Check WireGuard

```bash
sudo wg show
```

Interface:

```bash
ip addr show wg0
```

Routes:

```bash
ip route | grep wg
```

Configuration:

```bash
sudo cat /etc/wireguard/wg0.conf
```

Check service:

```bash
systemctl status wg-quick@wg0
```

---

# 21. Check DNS/Pi-hole-related configuration

Since your lab uses DNS infrastructure:

```bash
resolvectl status
```

```bash
cat /etc/resolv.conf
```

```bash
grep -RniE 'nameserver|search|domain' /etc/systemd /etc/NetworkManager 2>/dev/null
```

NetworkManager:

```bash
nmcli connection show
nmcli device show
```

---

# 22. Check logs

System logs:

```bash
sudo journalctl --since "7 days ago"
```

Errors:

```bash
sudo journalctl -p err..alert --since "7 days ago"
```

Warnings:

```bash
sudo journalctl -p warning..alert --since "7 days ago"
```

Kernel:

```bash
sudo journalctl -k --since "7 days ago"
```

Authentication:

```bash
sudo journalctl _SYSTEMD_UNIT=ssh.service --since "30 days ago"
```

Boot history:

```bash
journalctl --list-boots
```

Current boot:

```bash
sudo journalctl -b
```

Previous boot:

```bash
sudo journalctl -b -1
```

---

# 23. Search logs for suspicious activity

```bash
sudo journalctl --since "30 days ago" | \
grep -Ei 'failed|invalid|authentication|sudo|permission denied|segfault|malware|virus|rootkit|backdoor|reverse shell'
```

Network-related events:

```bash
sudo journalctl --since "7 days ago" | \
grep -Ei 'network|connection|dns|ssh|wireguard|iptables|nft'
```

---

# 24. Check disk and mounted filesystems

```bash
lsblk -f
```

```bash
df -hT
```

```bash
mount
```

```bash
findmnt
```

Check unusual mounts:

```bash
findmnt -t tmpfs,overlay,fuse,sshfs,nfs,cifs
```

Check `/etc/fstab`:

```bash
cat /etc/fstab
```

---

# 25. Check USB devices

```bash
lsusb
```

Detailed:

```bash
sudo lsusb -v
```

Recent kernel USB events:

```bash
sudo journalctl -k | grep -i usb
```

---

# 26. Check persistence through desktop autostart

```bash
find ~/.config/autostart -type f -ls 2>/dev/null
```

System-wide:

```bash
sudo find /etc/xdg/autostart -type f -ls 2>/dev/null
```

Inspect:

```bash
cat ~/.config/autostart/* 2>/dev/null
```

---

# 27. Check capabilities

Find binaries with Linux capabilities:

```bash
sudo getcap -r / 2>/dev/null
```

This is particularly useful for spotting unexpected privilege paths.

---

# 28. Check suspicious deleted files

```bash
sudo lsof +L1
```

Focus on processes:

```bash
sudo lsof +L1 | grep -Ei 'deleted|tmp|shm'
```

---

# 29. Check outbound connections

Current connections:

```bash
sudo ss -tpn
```

Resolve process information:

```bash
sudo lsof -i -n -P
```

TCP:

```bash
sudo lsof -iTCP -n -P
```

UDP:

```bash
sudo lsof -iUDP -n -P
```

---

# 30. Check installed services

```bash
systemctl list-units --type=service
```

All installed service definitions:

```bash
systemctl list-unit-files --type=service
```

Look for unusual names:

```bash
systemctl list-unit-files --type=service | \
grep -Ei 'backdoor|remote|proxy|tunnel|shell|agent|update|sync'
```

Don't assume a matching name is malicious—investigate the service file.

---

# 31. Check system timers

```bash
systemctl list-timers --all
```

User timers:

```bash
systemctl --user list-timers --all
```

---

# 32. Check SSH known hosts

```bash
cat ~/.ssh/known_hosts
```

```bash
sudo cat /root/.ssh/known_hosts 2>/dev/null
```

---

# 33. Check network configuration changes

```bash
nmcli connection show
```

```bash
sudo find /etc/NetworkManager -type f -mtime -30 -ls
```

```bash
sudo find /etc/systemd/network -type f -ls 2>/dev/null
```

---

# 34. Check for rootkits

Install and run `rkhunter`:

```bash
sudo apt update
sudo apt install rkhunter
```

Update:

```bash
sudo rkhunter --update
```

Check:

```bash
sudo rkhunter --check
```

View warnings:

```bash
sudo rkhunter --list warnings
```

You can also use Lynis:

```bash
sudo apt install lynis
```

Run:

```bash
sudo lynis audit system
```

---

# 35. ClamAV malware scan

Install:

```bash
sudo apt install clamav clamav-daemon
```

Update signatures:

```bash
sudo freshclam
```

Scan your home directory first:

```bash
clamscan -r -i ~
```

Full filesystem scan:

```bash
sudo clamscan -r -i \
--exclude-dir=/proc \
--exclude-dir=/sys \
--exclude-dir=/dev \
--exclude-dir=/run \
/
```

This can take a **very long time**.

---

# 36. Check file hashes

For important suspicious files:

```bash
sha256sum /path/to/file
```

For example:

```bash
sha256sum ~/.bashrc
```

Compare hashes of critical binaries:

```bash
sha256sum /bin/bash
sha256sum /usr/bin/sudo
sha256sum /usr/bin/ssh
```

Package verification with `debsums` is generally more useful than manually hashing everything.

---

# 37. Check kernel command line

```bash
cat /proc/cmdline
```

Bootloader configuration:

```bash
sudo cat /etc/default/grub
```

---

# 38. Check suspicious aliases/functions

```bash
alias
```

```bash
declare -F
```

```bash
grep -RniE 'alias|function' ~/.bashrc ~/.zshrc ~/.profile 2>/dev/null
```

---

# 39. Collect everything into a forensic report

You can create a **read-only collection script** that doesn't kill processes or modify the system:

```bash
mkdir -p "$CASE"

{
echo "===== DATE ====="
date -u

echo "===== HOST ====="
hostnamectl

echo "===== KERNEL ====="
uname -a

echo "===== USERS ====="
who
w

echo "===== LAST LOGINS ====="
last -a

echo "===== FAILED LOGINS ====="
sudo lastb -a 2>/dev/null

echo "===== PROCESSES ====="
ps auxf

echo "===== LISTENING PORTS ====="
sudo ss -lntup

echo "===== NETWORK ====="
ip addr
ip route
ip neigh

echo "===== DNS ====="
resolvectl status

echo "===== SYSTEMD SERVICES ====="
systemctl --type=service --state=running

echo "===== FAILED SERVICES ====="
systemctl --failed

echo "===== TIMERS ====="
systemctl list-timers --all

echo "===== SUID ====="
sudo find / -xdev -type f -perm -4000 -ls 2>/dev/null

echo "===== SGID ====="
sudo find / -xdev -type f -perm -2000 -ls 2>/dev/null

echo "===== CAPABILITIES ====="
sudo getcap -r / 2>/dev/null

echo "===== CRON ====="
sudo cat /etc/crontab
crontab -l 2>/dev/null

echo "===== MOUNTS ====="
findmnt

echo "===== DOCKER ====="
docker ps -a 2>/dev/null
docker images 2>/dev/null
docker network ls 2>/dev/null

echo "===== KERNEL LOG ====="
sudo journalctl -k --since "7 days ago"

echo "===== ERROR LOG ====="
sudo journalctl -p err..alert --since "7 days ago"

} 2>&1 | tee "$CASE/forensic-report.txt"
```

Then:

```bash
ls -lh "$CASE"
```

---

## 40. Compress the evidence

After collection:

```bash
tar -czf "${CASE}.tar.gz" "$CASE"
```

Hash it:

```bash
sha256sum "${CASE}.tar.gz" | tee "${CASE}.sha256"
```

Check:

```bash
cat "${CASE}.sha256"
```

---

## What I'd prioritize on your Kali

Because your machine is being used for **Kali/VMware, GOAD, Docker/Mythic, WireGuard and security labs**, don't treat every unusual process or connection as malicious. Those environments naturally generate a lot of unusual activity.

I'd investigate in this order:

1. **Unknown logged-in users**
    
2. **Unknown UID 0 accounts**
    
3. **Unexpected SSH keys**
    
4. **Unexpected listening ports**
    
5. **Unknown outbound connections**
    
6. **Unexpected systemd services/timers**
    
7. **Cron persistence**
    
8. **Recently modified `/etc` files**
    
9. **SUID/SGID/capability anomalies**
    
10. **Deleted executables still running**
    
11. **Unexpected Docker containers/images**
    
12. **Package integrity**
    
13. **Authentication logs**
    
14. **Kernel logs**
    
15. **Rootkit/malware scans**
    

If you want a **single automated `kali-forensic.sh` script** that runs these checks, timestamps everything, separates `CRITICAL / WARNING / INFO`, and produces a clean report without modifying or “cleaning” your system, I can build that for you.