# RHCSA Practice Lab Scenarios: Instructions for the Exam Taker

This document outlines the tasks required to complete the RHCSA-focused lab scenario on **VM1** and **VM2**. Assume you are the `sys_admin` user for all tasks unless explicitly stated otherwise.

## 🔑 Lab Credentials

| Host | User | Password | Initial Status |
| :--- | :--- | :--- | :--- |
| **VM1** (`<VM1_IP>`) | `sys_admin` | `pa$$word` | Functional |
| **VM2** (`<VM2_IP>`) | `sys_admin` | **Locked** | **Locked (Fix required)** |
| **VM1 Root** | `root` | `pa$$word` | Functional |
| **VM2 Root** | `root` | **Unknown** | **Unknown (Recovery required)** |
| **VM2** (`<VM2_IP>`) | `henry` | `testpass` | Functional, standard user |

## 🎯 VM1 Scenario: Database Infrastructure & Target Management

**Goal:** Configure the system to host a new database installation, implement required storage, and adjust system behavior.

### Storage Configuration (LVM)
The system has three virtual disks: `/dev/vda` (OS), `/dev/vdb` (5GB for database), and `/dev/vdc` (1GB for swap).
1.  Verify the existence of the free disks, `/dev/vdb` and `/dev/vdc`.
2.  Create a **Physical Volume (PV)** using only **half of the available space** on `/dev/vdb`.
3.  Create a **Volume Group (VG)** named `db_vg` using the new PV.
4.  Create a **Logical Volume (LV)** named `db_lv` inside `db_vg` that is exactly **1GB** in size.
5.  Format the new `db_lv` using the **XFS** filesystem.
6.  The database mount point is already created at `/opt/database`. Ensure this directory is owned by `root:db_users` and has **sticky group permissions** (`g+s`) set.
7.  Mount the new `/dev/db_vg/db_lv` volume to `/opt/database` and ensure it mounts **persistently after reboot** using the volume's **UUID**.

### System Boot and Management
1.  Change the system's **default boot target** from `graphical.target` to `multi-user.target`.
2.  **Reboot the system** to verify the persistent mount and the new boot target.
3.  After reboot, extend the `db_lv` Logical Volume (using the available free space in `db_vg`) to a final size of **2GB**. Resize the filesystem accordingly.

### General Objectives Check
1.  **Scripting:** Create a simple shell script (`/usr/local/bin/date_rename.sh`) that takes a filename as an argument (`$1`) and uses a loop to rename all files in the current directory that match the input argument, appending the current date (`YYYYMMDD`) to their name.
2.  **Redirection/Grep:** Output all lines that contain "ERROR:" from `/tmp/log_data.txt` to a file named `errors_only.txt` using input redirection. Then, use `grep` with a regular expression to count all lines containing the date format `2025/11/29`.
3.  **Tuning:** Permanently switch the active tuning profile to `latency-performance`.
4.  **Archive:** Create a compressed archive of `/data/backup_source` using `bzip2`, name it `backup.tar.bz2`, and verify the original files are intact.

## ⚠️ VM2 Scenario: Critical System Failure & Troubleshooting

**Goal:** The system is in a broken state due to a failed maintenance script. You must first gain access, then fix the errors, and restore network and web services.

### Recovery and User Access
1.  Attempt to boot **VM2**. Note the boot failure (due to `/etc/fstab` error).
2.  **Interrupt the boot process** (boot menu/GRUB) and enter emergency mode to gain access to the root filesystem.
3.  **Recover the unknown root password** and set it to `pa$$word`.
4.  **Fix the boot error:** Correct the malicious entry in `/etc/fstab`.
5.  Reboot the system into `multi-user.target` manually to verify the fix.
6.  Using your newly recovered root privileges, **unlock the `sys_admin` user account**.
7.  Log out and log back in as `sys_admin` using the password `pa$$word`.

### Networking and Services Repair
1.  **Correct the hostname:** Set the system hostname to `vm2.lab.example.com`.
2.  **Correct the IP address:** Fix the incorrect static IPv4 configuration to the required `192.168.1.246` and ensure it starts automatically at boot.
3.  **Install/Configure HTTPD:** The web server is installed but failing to serve content on port 82.
    * Troubleshoot and fix the `httpd` service so it runs on **port 82**.
    * The file `/var/www/html/index.html` has incorrect permissions. Correct them so the web server can read it.
    * **SELinux:** Use `semanage` to configure the port label and the file context on `/var/www/html/success.html` so the web server can serve the file without errors.
    * Update the firewall to permanently allow HTTP access on the correct port (82/tcp).
    * **Verification:** Access `http://vm2.lab.example.com:82/success.html` from a browser on your host machine. The page must display **"Great Success!"**

### Data Sharing and Scheduling
1.  **Create Production Users:** Create a group named `production` and two users, `prod_user1` and `prod_user2`, ensuring their UIDs/GIDs **match** the users created on VM1 (i.e., you may need to use `usermod -u` and `groupmod -g`).
2.  **Autofs NFS Mount:** Configure `autofs` to automatically mount the shared NFS directory (`/opt/nfs`) from **VM1** onto `/home/nfs` on **VM2** for the `production` group.

### General Objectives Check
1.  **Process Management:** The system is a CPU-intensive process that is unnecessarily consuming resources. Identify this process using standard management tools (top, ps, etc.) and terminate it using the appropriate signal (kill).
2.  **Scripting/Scheduling:** Create the shell script /usr/local/bin/log_disk_usage.sh. The script should log the disk usage of the / partition (e.g., using df -h) to the system journal.
3. **Scheduling:**  Create a systemd timer unit that executes the script created in step 2 every Sunday at 04:00 AM. Ensure the service and timer are enabled and running.
4.  **Software Management:** Install a basic package (e.g., mlocate) using the configured RPM repository. Then, as the user henry (without sudo), manually configure a new Flatpak remote named local-app-repo using the URL http://{{ hostvars['repo']['ansible_host'] }}/flatpak_repo. Finally, install the application com.example.HelloWorld so that only henry can use it.