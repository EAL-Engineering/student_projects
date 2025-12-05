# Phase 1, Task 2: Endian Firewall Configuration (DHCP & PXE Handoff)

**Goal:** Configure Endian to provide IPs to all nodes but redirect Compute Nodes (02-04) to boot from the Head Node (01).
**Reference:** Project Plan Section 4 (Phase 1) & Section 2 (Network Fabric)

## 1. Prerequisites
Gather the following before starting:
* **Node 01 IP (Next-Server):** e.g., `10.10.10.1` (This node hosts the OS images).
* **MAC Addresses:** Required for all 4 nodes (Port 1 / IPMI usually).
* **Boot Filename:** `ipxe.efi` (Standard for Warewulf 4/UEFI).

## 2. Access Configuration
1.  Log in to **Endian Firewall Web Interface**.
2.  Navigate to **Services** -> **DHCP Server**.
3.  Select the **Green Interface** (or the specific zone for the cluster).

## 3. Configure Static Leases (Fixed IPs)
*Note: Do not use dynamic ranges for cluster nodes. Slurm requires fixed DNS/IP mapping.*

**A. Add Head Node (Node 01)**
1.  Click **Add new fixed lease**.
2.  **MAC Address:** `<MAC_of_Node01>`
3.  **IP Address:** `<Planned_IP_for_Node01>`
4.  **Remark:** `Head Node / TFTP Server`
5.  **Next-Server:** (Leave Blank - it boots from local disk)
6.  **Filename:** (Leave Blank)

**B. Add Compute Nodes (Nodes 02-04)**
1.  Click **Add new fixed lease**.
2.  **MAC Address:** `<MAC_of_Node02>`
3.  **IP Address:** `<Planned_IP_for_Node02>`
4.  **Remark:** `Compute Node 02`
5.  **Next-Server:** `10.10.10.1` (IP of Node 01)
6.  **Filename:** `ipxe.efi`
*(Repeat for Nodes 03 and 04)*

## 4. Alternative: Global PXE Redirection
*Use this if you prefer to set the boot options for the entire subnet rather than per-node.*

1.  Scroll to **Settings** -> **Custom Configuration** (or "Extra Options").
2.  Paste the following:
    
    # Option 66: IP of the TFTP Server (Head Node)
    option tftp-server-name "10.10.10.1";
    
    # Option 67: The Boot File (served by Warewulf)
    option bootfile-name "ipxe.efi";

## 5. Apply & Verify
1.  Click **Save**.
2.  Click **Apply** (top banner) to restart the DHCP service.
3.  **Verification:**
    * Do not boot compute nodes yet.
    * From a Linux laptop on the same network, run: `sudo nmap --script broadcast-dhcp-discover`
    * Verify the output shows `Server Identifier` (Endian) and `Boot File Name` (ipxe.efi).