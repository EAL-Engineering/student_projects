# Phase 1, Task 2: Endian Firewall Configuration (DHCP & PXE Handoff)

**Goal:** Configure Endian to provide IPs to all nodes but redirect Compute Nodes (02-04) to boot from the Head Node (01).
**Reference:** Project Plan Section 4 (Phase 1) & Section 2 (Network Fabric)

## 1. Network Parameters & Assignment
**Network Topology:**
* **Subnet:** `10.0.0.0`
* **Netmask:** `255.255.0.0` (/16)
* **Gateway / DNS:** `10.0.0.254` (Endian Firewall)
* **Dynamic Pool:** `10.0.0.175` - `10.0.0.250` (Do not use for cluster)

**New Cluster IP Allocation (90 Series):**
| Role | Hostname | IP Address | Boot Method |
| :--- | :--- | :--- | :--- |
| **Head Node** | `node01` (Node A) | **10.0.0.91** | Local Disk (No PXE Redirection) |
| Compute Node | `node02` | **10.0.0.92** | PXE -> Redirect to 10.0.0.91 |
| Compute Node | `node03` | **10.0.0.93** | PXE -> Redirect to 10.0.0.91 |
| Compute Node | `node04` | **10.0.0.94** | PXE -> Redirect to 10.0.0.91 |

## 2. Access Configuration
1.  Log in to the **Endian Firewall Web Interface** (`https://10.0.0.254:10443`).
2.  Navigate to **Services** -> **DHCP Server**.
3.  Click the **Fixed leases** tab.

## 3. Configure Static Leases (Fixed IPs)
*Note: You must click the "Advanced options" dropdown to see the boot settings.*

**A. Add Head Node (Node 01 / Node A)**
1.  Click **Add a fixed lease**.
2.  **MAC address:** `AC:1F:6B:CB:3C:3B`
3.  **IP address:** `10.0.0.91`
4.  **Remark:** `Head Node (Node A) - Data`
5.  Click **Advanced options**:
    * **Next address:** (Leave Blank)
    * **Filename:** (Leave Blank)
6.  Click **Add**.

**B. Add Compute Nodes (Nodes 02-04)**
*(Waiting for MAC addresses from Node B, C, and D images)*

1.  Click **Add a fixed lease**.
2.  **MAC address:** `[Pending Node B MAC]`
3.  **IP address:** `10.0.0.92`
4.  **Remark:** `Compute Node 02`
5.  Click **Advanced options**:
    * **Next address:** `10.0.0.91`
    * **Filename:** `ipxe.efi`
6.  Click **Add**.

*(Repeat for Nodes 03 and 04 once MACs are confirmed)*

## 4. Apply & Verify
1.  Click **Apply** (top banner) to restart the DHCP service.
2.  **Verification:**
    * From a network machine: `ping 10.0.0.91` (after Node 01 boot).
    * Check `Next-Server` (Option 66) is `10.0.0.91` for the compute nodes.