# Phase 1, Step 3: Head Node OS Installation

* **File:** `03-almalinux-install.md`
* **Date:** December 5, 2025
* **Target:** Node 01 (Head Node)
* **OS:** AlmaLinux 9.x
* **Hardware:** Supermicro X10 "Twin" (2x Xeon E5-2630 v4)

---

## 1. BIOS Configuration (Pre-Flight)
Before booting the installation media, ensure the Supermicro X10 BIOS is configured correctly to support modern OS features and future hardware expansion.

* **Boot Mode:** **UEFI** (Disable Legacy/CSM).
* **PCIe Configuration:** Enable **Above 4G Decoding**.
    * *Why:* Essential for addressing large memory spaces and critical if accelerators (GPUs/Xeon Phi) are added later.
* **SATA Mode:** **AHCI**.
    * *Note:* Do not use "Intel RST/RAID" unless you have explicitly configured a specific RAID array. Software RAID (MDADM) or LVM is preferred for this setup.

---

## 2. Installation Procedure
Boot from the AlmaLinux 9 installation media. Select **"Install AlmaLinux 9"**.

### A. Localization
* **Language:** English (United States).
* **Time & Date:** Select local timezone (e.g., New York).
    * *Tip:* Toggle **Network Time** to **ON** (top right). It may not sync immediately, but it ensures NTP is active post-install.

### B. Software Selection (Critical)
* **Base Environment:** **Minimal Install**.
* **Add-Ons:** None.
    * *Why:* OpenHPC is best built on a clean slate. "Server with GUI" installs unnecessary packages that consume resources and increase security risks.

### C. Partitioning (Installation Destination)
Select the primary OS drive and choose **Custom** storage configuration. Click **Done**.
**Scheme:** Standard Partition or LVM (LVM preferred).

| Mount Point | Size | Filesystem | Rationale |
| :--- | :--- | :--- | :--- |
| `/boot/efi` | 600 MiB | EFI System | Required for UEFI boot. |
| `/boot` | 1 GiB | XFS | Kernel storage. |
| `swap` | 8 GiB | Swap | Safety net for RAM overflows. |
| `/` (Root) | **100 GiB** | XFS | **Larger than standard.** Warewulf stores compute node VNFS images in `/var/lib/warewulf`, which resides here. |
| `/home` | Remainder | XFS | This partition will be NFS exported to compute nodes for user data storage. |

### D. Network & Hostname
* **Hostname:** `node01` (Set in the bottom left text box and click Apply).
* **Interface Configuration:**
    * Identify the primary interface (likely `eno1`).
    * **Do not configure the LACP Bond yet.** The installer can struggle with LACP negotiation during setup.
    * Configure `eno1` with a temporary static IP (IPv4) to ensure internet access for updates.
    * **IPv4 Settings:** Manual -> Address, Netmask, Gateway, DNS (8.8.8.8).
    * **Activate:** Toggle the switch to **ON**.

### E. User Settings
* **Root Password:** Set a strong root password.
* **User Creation:** Create a primary admin user (e.g., `admin`).
    * Check **"Make this user administrator"** (Adds to wheel group for sudo access).

Click **Begin Installation**.

---

## 3. Post-Installation Configuration
After the installation completes, reboot the system and log in as `root`.

### A. System Update
Ensure the base OS kernel and security patches are current.
```bash
dnf -y update