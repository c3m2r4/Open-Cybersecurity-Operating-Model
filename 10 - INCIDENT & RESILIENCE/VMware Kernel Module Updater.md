# VMware Kernel Module Updater - Fix Runbook

## 🎯 Objective
Resolve the issue where VMware Workstation or VMware Player gets stuck in an endless loop, repeatedly asking to compile kernel modules (via the "VMware Kernel Module Updater" popup) after a system/kernel update on Kali Linux.

## 🔍 Symptoms
* You launch VMware and get a popup stating: *"Before you can run VMware, several modules must be compiled and loaded into the running kernel."*
* You click **Install**, authenticate with your password, and it appears to succeed (or fails silently).
* The next time you launch VMware, the exact same prompt appears again.

## 🧠 Root Cause
When Kali Linux updates the kernel and compiles new VMware modules (`vmmon` and `vmnet`), the system's strict default security settings (umask) restrict read permissions on the `/lib/modules/` directory. 

When you try to open VMware as a normal user, a startup script runs `modprobe` to check if the modules exist. Because your normal user is blocked from reading the directory, the script assumes the modules are missing and triggers the popup prompt again.

---

## 🛠️ Step-by-Step Fix

Whenever you update your Kali Linux kernel and encounter this issue, run the following commands in your terminal:

### Step 1: Force compile the VMware modules
Ensure the modules are built for your current kernel.
```bash
sudo vmware-modconfig --console --install-all
```

### Step 2: Rebuild module dependencies
Update the system's kernel module index so it registers the newly built VMware modules.
```bash
sudo depmod -a
```

### Step 3: Fix directory permissions (The Magic Step)
Grant read and execute permissions to the kernel modules directory so that your normal user account (and the VMware startup script) can verify the modules are installed.
```bash
sudo chmod -R a+rX /lib/modules/$(uname -r)
```

### Step 4: Restart VMware services
Ensure the background daemon is running cleanly with the newly loaded modules.
```bash
sudo /etc/init.d/vmware restart
```

---

## ⚡ Quick-Run Script

For convenience, you can copy and paste this entire block into your terminal to run all the steps automatically the next time this happens:

```bash
echo "Building VMware modules..." && \
sudo vmware-modconfig --console --install-all && \
echo "Updating module dependencies..." && \
sudo depmod -a && \
echo "Fixing restrictive folder permissions..." && \
sudo chmod -R a+rX /lib/modules/$(uname -r) && \
echo "Restarting VMware services..." && \
sudo /etc/init.d/vmware restart && \
echo "Done! You can now launch VMware."
```
