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
| **Head Node** | `node01` | **10.0.0.91** | Local Disk (No PXE Redirection) |
| Compute Node | `node02` | **10.0.0.92** | PXE -> Redirect to 10.0.0.91 |
| Compute Node | `node03` | **10.0.0.93** | PXE -> Redirect to 10.0.0.91 |
| Compute Node | `node04` | **10.0.0.94** | PXE -> Redirect to 10.0.0.91 |

## 2. Access Configuration
1.  Log in to the **Endian Firewall Web Interface** (`https://10.0.0.254:10443`).
2.  Navigate to **Services** -> **DHCP Server**.
3.  Select the **Green Interface**.

## 3. Configure Static Leases (Fixed IPs)

**A. Add Head Node (Node 01)**
1.  Click **Add new fixed lease**.
2.  **MAC Address:** `<MAC_of_Node01>`
3.  **IP Address:** `10.0.0.91`
4.  **Remark:** `New Cluster Head Node`
5.  **Next-Server:** (Leave Blank - boots locally)
6.  **Filename:** (Leave Blank)

**B. Add Compute Nodes (Nodes 02-04)**
1.  Click **Add new fixed lease**.
2.  **MAC Address:** `<MAC_of_Node02>`
3.  **IP Address:** `10.0.0.92`
4.  **Remark:** `New Cluster Node 02`
5.  **Next-Server:** `10.0.0.91` (Points to Head Node)
6.  **Filename:** `ipxe.efi`

*(Repeat for Node 03 at `10.0.0.93` and Node 04 at `10.0.0.94`, ensuring Next-Server is always `10.0.0.91`)*

## 4. Alternative: Global PXE Redirection
*If you prefer to set the boot options for the entire subnet (Warning: Ensure this doesn't conflict with other PXE devices on the 10.0.x.x network).*

1.  Scroll to **Settings** -> **Custom Configuration**.
2.  Paste the following:
    
    # Option 66: IP of the TFTP Server (New Head Node 01)
    option tftp-server-name "10.0.0.91";
    
    # Option 67: The Boot File (served by Warewulf on Node 01)
    option bootfile-name "ipxe.efi";

## 5. Apply & Verify
1.  Click **Save**.
2.  Click **Apply** (top banner) to restart the DHCP service.
3.  **Verification:**
    * From a machine on the `10.0.x.x` network, ping `10.0.0.91` (once Node 01 is up) to verify the address is free/active.
    * Run `sudo nmap --script broadcast-dhcp-discover` to ensure the `Next-Server` field reports `10.0.0.91`.