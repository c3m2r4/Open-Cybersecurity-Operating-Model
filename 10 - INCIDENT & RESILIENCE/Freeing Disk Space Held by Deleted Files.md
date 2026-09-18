# Runbook: Freeing Disk Space Held by Deleted Files

> [!NOTE]
> This runbook is designed to help you troubleshoot and resolve situations where you have deleted files, but the disk space has not been freed up. This typically occurs because a background process is still holding those files open.

## Background
When a file is deleted in Linux, the filesystem removes the link to the file. However, if a running process has that file open (such as a Virtual Machine, a database, or a web server), the OS will not actually free the disk blocks until that process closes the file or is terminated. 

## Symptoms
- You deleted large files (e.g., from `/mnt/vmstorage`).
- Running `df -h` still shows the disk as full or space as used.
- Running `du -sh` on the directory shows very little space used (since the files are hidden from the directory tree).

---

## Step 1: Check Current Disk Usage
Verify the overall disk space to confirm the space is still being consumed.

```bash
df -h
```
*Look for the `Use%` and `Avail` columns for your specific partition.*

## Step 2: Find Deleted Files Held by Processes
Use the `lsof` (List Open Files) command to find files that have been deleted from the directory tree but are still held open by processes. The `+L1` flag specifically filters for files with less than 1 link (i.e., deleted files).

```bash
sudo lsof +L1
```
*If you are looking for files on a specific mount point, you can grep for it:*
```bash
sudo lsof +L1 | grep /mnt/vmstorage
```

**Expected Output Example:**
```text
COMMAND      PID    USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
vmware-vm 496001  camara   75u   REG    8,1 52428800     0 6817732 /mnt/vmstorage/.../disk.vmdk (deleted)
```

## Step 3: Identify the Process
From the output in Step 2, identify the `PID` (Process ID) and the `COMMAND` holding the files open.
- **COMMAND:** The name of the process (e.g., `vmware-vm`).
- **PID:** The process ID (e.g., `496001`).

You can inspect what this process is doing before killing it:
```bash
ps -p <PID> -f
```

## Step 4: Terminate the Process
Once you have confirmed that the process can be safely closed (and any unsaved work in that process is accounted for), terminate it to release the file handles.

1. **Attempt a graceful shutdown first:**
   ```bash
   kill <PID>
   ```
2. **Forcefully kill if it does not respond:**
   ```bash
   kill -9 <PID>
   ```

> [!CAUTION]
> Using `kill -9` will force-quit the process immediately. If this is a critical system service or database, it could lead to corruption. Ensure it is safe to terminate (e.g., an orphaned VM process).

## Step 5: Verify Space Recovery
Run the disk usage command again to confirm the space has been successfully reclaimed.

```bash
df -h
```
The `Avail` column should now reflect the newly freed space.
