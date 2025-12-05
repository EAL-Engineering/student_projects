# Phase 1, Step 3: Head Node OS Installation

* **File:** `03-almalinux-install.md`
* **Date:** December 5, 2025
* **Target:** Node 01 (Head Node)
* **OS:** AlmaLinux 9.x
* **Hardware:** Supermicro X10 "Twin" (2x Xeon E5-2630 v4)
* **Storage Config:** Dual-Drive (SATA Boot + NVMe Data)

---

## 1. BIOS Configuration (Pre-Flight)
Before booting the installation media, ensure the Supermicro X10 BIOS is configured correctly.

* **Boot Mode:** **UEFI** (Disable Legacy/CSM).
* **PCIe Configuration:** Enable **Above 4G Decoding** (Critical for NVMe support).
* **SATA Mode:** **AHCI**.
    * *Note:* Do not use "Intel RST/RAID".

---

## 2. Installation Procedure
Boot from the AlmaLinux 9 installation media. Select **"Install AlmaLinux 9"**.

### A. Localization
* **Language:** English (United States).
* **Time & Date:** Select local timezone (e.g., New York).
    * *Tip:* Toggle **Network Time** to **ON** (top right).

### B. Software Selection (Critical)
* **Base Environment:** **Minimal Install**.
* **Add-Ons:** None.

### C. Partitioning (Installation Destination)
Select **BOTH** the SATA SSD and the NVMe drive. Choose **Custom** storage configuration. Click **Done**.

**Strategy:**
* **Drive 1 (SATA):** Holds the OS, Logs, and Warewulf Images.
* **Drive 2 (NVMe):** Dedicated strictly to `/home` to maximize NFS throughput.

**Partition Layout:**

| Drive | Mount Point | Size | Filesystem | Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **SATA** | `/boot/efi` | 600 MiB | EFI System | UEFI Boot loader. |
| **SATA** | `/boot` | 1 GiB | XFS | Kernel storage. |
| **SATA** | `swap` | 8 GiB | Swap | Safety net for RAM overflows. |
| **SATA** | `/` (Root) | **Max Allowed** | XFS | Uses the rest of the SATA drive. ample space for OS + Warewulf images (`/var/lib/warewulf`). |
| **NVMe** | `/home` | **Max Allowed** | XFS | Uses 100% of the NVMe drive. Ensures user I/O saturates the 10GbE link. |

*Note: If the installer asks, ensure the Boot Loader is installed to the SATA drive.*

### D. Network & Hostname
* **Hostname:** `node01` (Set in the bottom left text box and click Apply).
* **Interface Configuration:**
    * Identify the primary interface (likely `eno1`).
    * **Do not configure the LACP Bond yet.**
    * Configure `eno1` with a temporary static IP (IPv4).
    * **Activate:** Toggle the switch to **ON**.

### E. User Settings
* **Root Password:** Set a strong root password.
* **User Creation:** Create a primary admin user.
    * **Username:** `eal`
    * Check **"Make this user administrator"** (Adds to wheel group for sudo access).

Click **Begin Installation**.

---

## 3. Post-Installation Configuration
After the installation completes, reboot the system and log in as `root`.

### A. System Update
```bash
dnf -y update