# Phase 1, Task 2: Endian Firewall Configuration (DHCP & PXE Handoff)

* **Goal:** Configure Endian to provide IPs to all nodes but redirect Compute Nodes (02-04) to boot from the Head Node (01).
* **Reference:** Project Plan Section 4 (Phase 1) & Section 2 (Network Fabric)

## 1. Network Parameters & Assignment
**Network Topology:**
* **Subnet:** `10.0.0.0`
* **Netmask:** `255.255.0.0` (/16)
* **Gateway / DNS:** `10.0.0.254` (Endian Firewall)

**Cluster IP Allocation (90 Series):**
| Role | Hostname | IP Address | MAC Address (10GbE Port 1) | Boot Method |
| :--- | :--- | :--- | :--- | :--- |
| **Head Node** | `node01` (Node A) | **10.0.0.91** | `AC:1F:6B:C4:C2:58` | Local Disk |
| Compute Node | `node02` (Node B) | **10.0.0.92** | `AC:1F:6B:C4:C4:66` | PXE -> Redirect |
| Compute Node | `node03` (Node C) | **10.0.0.93** | `AC:1F:6B:59:A0:B6` | PXE -> Redirect |
| Compute Node | `node04` (Node D) | **10.0.0.94** | `AC:1F:6B:5A:BD:04` | PXE -> Redirect |

## 2. Access Configuration
1.  Log in to the **Endian Firewall Web Interface** (`https://10.0.0.254`).
2.  Navigate to **Services** -> **DHCP Server**.
3.  Click the **Fixed leases** tab.

## 3. Configure Static Leases (Fixed IPs)
*Note: You must click the "Advanced options" dropdown to see the boot settings.*

**A. Add Head Node (Node 01 / Node A)**
1.  Click **Add a fixed leases**.
2.  **MAC address:** `AC:1F:6B:C4:C2:58`
3.  **IP address:** `10.0.0.91`
4.  **Remark:** `Head Node (Node A) - 10GbE`
5.  Click **Advanced options**:
    * **Next address:** (Leave Blank)
    * **Filename:** (Leave Blank)
6.  Click **Add**.

**B. Add Compute Node 02 (Node B)**
1.  Click **Add a fixed leases**.
2.  **MAC address:** `AC:1F:6B:C4:C4:66`
3.  **IP address:** `10.0.0.92`
4.  **Remark:** `Compute Node 02 (Node B) - 10GbE`
5.  Click **Advanced options**:
    * **Next address:** `10.0.0.91`
    * **Filename:** `ipxe.efi`
6.  Click **Add**.

**C. Add Compute Node 03 (Node C)**
1.  Click **Add a fixed leases**.
2.  **MAC address:** `AC:1F:6B:59:A0:B6`
3.  **IP address:** `10.0.0.93`
4.  **Remark:** `Compute Node 03 (Node C) - 10GbE`
5.  Click **Advanced options**:
    * **Next address:** `10.0.0.91`
    * **Filename:** `ipxe.efi`
6.  Click **Add**.

**D. Add Compute Node 04 (Node D)**
1.  Click **Add a fixed leases**.
2.  **MAC address:** `AC:1F:6B:5A:BD:04`
3.  **IP address:** `10.0.0.94`
4.  **Remark:** `Compute Node 04 (Node D) - 10GbE`
5.  Click **Advanced options**:
    * **Next address:** `10.0.0.91`
    * **Filename:** `ipxe.efi`
6.  Click **Add**.

## 4. Apply & Verify
1.  Click **Apply** (top banner) to restart the DHCP service.
2.  **Verification:**
    * From a network machine: `ping 10.0.0.91` (once Node 01 OS is installed).
    * Check `Next-Server` (Option 66) is `10.0.0.91` for the compute nodes.