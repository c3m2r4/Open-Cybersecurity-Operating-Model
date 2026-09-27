# Bluetooth Operations Runbook

This runbook covers how to utilize your Bluetooth adapter for everything from basic device discovery to advanced scanning and reconnaissance, particularly focusing on tools available in Kali Linux.

> [!TIP]
> Ensure your Bluetooth adapter is unblocked before starting any operations by running `sudo rfkill unblock bluetooth`.

---

## 1. Interface Management
Before scanning, you need to ensure the physical interface is up and recognized.

**Check adapter status:**
```bash
sudo hciconfig -a
```

**Bring the adapter UP or DOWN:**
```bash
sudo hciconfig hci0 up
sudo hciconfig hci0 down
```

**Reset the adapter (useful if it hangs):**
```bash
sudo hciconfig hci0 reset
```

---

## 2. Standard Discovery (`bluetoothctl`)
`bluetoothctl` is the modern, interactive frontend for the BlueZ stack. It is best used for everyday pairing, trusting, and standard discovery.

**Enter the interactive shell:**
```bash
sudo bluetoothctl
```
*(Once inside the `[bluetooth]#` prompt, you can run the following commands)*

- `power on` : Ensure controller is powered
- `scan on` : Start scanning for nearby Classic and LE devices
- `devices` : List all devices discovered so far
- `info <MAC_ADDRESS>` : Get detailed information about a specific device
- `scan off` : Stop scanning
- `quit` : Exit the shell

---

## 3. Low Energy (LE) Scanning
Bluetooth Low Energy (BLE) devices (like smartwatches, IoT devices, and beacons) operate differently than Bluetooth Classic. 

**Basic LE Scan:**
```bash
sudo hcitool lescan
```
*(Press `Ctrl+C` to stop. This will flood the terminal with MAC addresses and broadcast names).*

**Advanced Management (`btmgmt`):**
`btmgmt` is a powerful tool to interact directly with the Bluetooth management API.
```bash
sudo btmgmt
```
Inside the `btmgmt` shell:
- `find -l` : Scan for LE devices
- `find -b` : Scan for BR/EDR (Classic) devices
- `info` : Show detailed controller capabilities

---

## 4. Advanced Reconnaissance & Enumeration (Kali Linux)
Since you are on Kali Linux, you have access to powerful diagnostic and offensive Bluetooth tools.

### A. Service Discovery (`sdptool`)
Find out what services (Audio, OBEX, Serial, etc.) a target Classic Bluetooth device is running.
```bash
sudo sdptool browse <MAC_ADDRESS>
```

### B. L2CAP Pinging (`l2ping`)
Verify if a target Bluetooth device is in range and reachable by sending an L2CAP echo request (similar to ICMP ping).
```bash
sudo l2ping -i hci0 <MAC_ADDRESS>
```

### C. Continuous Logging (`bluelog`)
`bluelog` is a highly configurable Bluetooth scanner designed to run quietly in the background and log discovered devices to a file.
```bash
sudo bluelog -i hci0 -n -o /tmp/bluetooth_devices.log
```
- `-n` : Resolve device names
- `-o` : Output file

### D. Bettercap (Advanced BLE)
`bettercap` is a powerful framework that includes an excellent BLE module for enumeration and character reading/writing.
```bash
sudo bettercap -eval "ble.recon on"
```
Once inside bettercap, you can type `help ble` to see modules for writing to BLE characteristics, sniffing traffic, and enumerating GATT profiles.

---

## 5. Troubleshooting & Diagnostics

> [!WARNING]
> If tools like `hcitool` return `Device is not available: No such device`, it means another process (like `bluetoothd`) is actively locking the adapter for scanning. 

**Monitor raw Bluetooth traffic (HCI Dump):**
If you want to see the raw packets being sent and received by your dongle (excellent for debugging connections or seeing raw broadcast payloads):
```bash
sudo hcidump -i hci0 -X -V
```
- `-X` : Hex dump payloads
- `-V` : Verbose protocol decoding
