# WiFi Adapter Runbook
### ZTOP ZT9101 USB Adapter — Kali Linux 7.1.5

> **Interface:** `wlan0` | **Driver:** `zt9101_ztopmac_usb` | **TX Power:** 12 dBm

---

## 1. Load the Driver (Manual)

Do this after every reboot until you install it permanently (see Section 9).

```bash
sudo insmod ~/driver_wifi_ztopinc/src/zt9101_ztopmac_usb.ko
```

Confirm it loaded:
```bash
lsmod | grep zt
# Expected: zt9101_ztopmac_usb  659456  0
```

---

## 2. Bring the Interface Up / Down

```bash
# Turn ON
sudo ip link set wlan0 up

# Turn OFF
sudo ip link set wlan0 down
```

Check current state:
```bash
ip link show wlan0
# Look for: UP or DOWN in the flags
```

---

## 3. Scan for Networks

```bash
sudo iw dev wlan0 scan
```

Compact view (SSID + signal only):
```bash
sudo iw dev wlan0 scan | grep -E "SSID:|signal:"
```

Sort by signal strength (strongest first):
```bash
sudo iw dev wlan0 scan 2>/dev/null | awk '
  /BSS /       { bss=$2 }
  /SSID:/      { ssid=$2 }
  /signal:/    { print $2, ssid, bss }
' | sort -rn
```

---

## 4. Connect to a Network

### WPA2 (most home/office networks)

```bash
# Step 1 — create a config file
wpa_passphrase "YOUR_SSID" "YOUR_PASSWORD" | sudo tee /etc/wpa_supplicant/wlan0.conf

# Step 2 — connect
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wlan0.conf

# Step 3 — get an IP address
sudo dhclient wlan0
```

### Open network (no password)

```bash
sudo iw dev wlan0 connect "SSID_NAME"
sudo dhclient wlan0
```

### Check connection status

```bash
iw dev wlan0 link
# Shows: SSID, BSSID, signal, TX/RX rates
```

---

## 5. Disconnect

```bash
# Release IP
sudo dhclient -r wlan0

# Disconnect from AP
sudo iw dev wlan0 disconnect

# Kill wpa_supplicant
sudo killall wpa_supplicant
```

---

## 6. Check Signal & Link Quality

```bash
# Live signal monitor (updates every 1s)
watch -n1 "iw dev wlan0 link && cat /proc/net/wireless"
```

```bash
# One-shot detailed info
iwconfig wlan0
```

Signal guide:
| dBm | Quality |
|-----|---------|
| > -50 | Excellent |
| -50 to -60 | Good |
| -60 to -70 | Fair |
| -70 to -80 | Weak |
| < -80 | Very poor |

---

## 7. Monitor Mode (for packet capture / Kali tools)

```bash
# Enable monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Verify
iwconfig wlan0
# Should show: Mode:Monitor
```

```bash
# Switch back to managed (normal) mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up
```

> **Note:** Tools like `airodump-ng`, `Wireshark`, `hostapd` require monitor mode.

---

## 8. Use with Kali Tools

### Airodump-ng (network capture)
```bash
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
```

### Aircrack-ng suite
```bash
# Capture handshake
sudo airodump-ng -c CHANNEL --bssid TARGET_BSSID -w capture wlan0mon
```

### Hostapd (Access Point mode)
```bash
# The driver supports AP mode (CFG_ENABLE_AP_MODE is compiled in)
# Create /etc/hostapd/hostapd.conf then:
sudo hostapd /etc/hostapd/hostapd.conf
```

---

## 9. Make Driver Load Automatically on Boot

Run these **once** to install permanently:

```bash
cd ~/driver_wifi_ztopinc/src

# Install the .ko to the kernel module tree
sudo make install

# Rebuild module dependency map
sudo depmod -a

# Tell the system to load it at boot
echo "zt9101_ztopmac_usb" | sudo tee /etc/modules-load.d/ztop-wifi.conf
```

Verify after next reboot:
```bash
lsmod | grep zt
```

---

## 10. Rebuild the Driver (after kernel update)

If Kali updates the kernel, the driver needs to be recompiled:

```bash
cd ~/driver_wifi_ztopinc/src
make clean
make
sudo make install
sudo depmod -a
```

---

## 11. Unload the Driver

```bash
sudo rmmod zt9101_ztopmac_usb
```

---

## 12. Troubleshooting

### Interface doesn't appear after insmod
```bash
sudo dmesg | grep -i "zt\|wlan\|usb" | tail -20
lsusb | grep -i "350b\|ztop"   # Check USB device is detected
```

### wlan0 stays DOWN after `ip link set up`
```bash
# Check for rfkill (hardware/software block)
rfkill list
sudo rfkill unblock wifi
```

### No IP after connecting
```bash
sudo dhclient -v wlan0
# or
sudo systemctl restart NetworkManager
```

### NetworkManager conflicts with manual wpa_supplicant
```bash
# Temporarily stop NetworkManager
sudo systemctl stop NetworkManager

# ... do your manual connection ...

# Re-enable when done
sudo systemctl start NetworkManager
```

### Check driver debug logs
```bash
sudo dmesg | grep -E "\[E\]|\[W\]" | tail -30   # errors and warnings only
sudo dmesg | grep -i zt | tail -40               # all driver messages
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Load driver | `sudo insmod ~/driver_wifi_ztopinc/src/zt9101_ztopmac_usb.ko` |
| Interface up | `sudo ip link set wlan0 up` |
| Scan | `sudo iw dev wlan0 scan` |
| Connect WPA2 | `sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wlan0.conf` |
| Get IP | `sudo dhclient wlan0` |
| Check link | `iw dev wlan0 link` |
| Monitor mode | `sudo iw dev wlan0 set type monitor` |
| Managed mode | `sudo iw dev wlan0 set type managed` |
| Disconnect | `sudo iw dev wlan0 disconnect && sudo dhclient -r wlan0` |
| Unload driver | `sudo rmmod zt9101_ztopmac_usb` |
| Signal monitor | `watch -n1 "iw dev wlan0 link"` |
