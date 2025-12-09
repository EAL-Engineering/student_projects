# Phase 1, Step 4: Head Node Network Bonding

* **File:** `04-network-bonding.md`
* **Date:** December 9, 2025
* **Target:** Node 01 (Head Node)
* **OS:** AlmaLinux 9.x
* **Goal:** Aggregate two 10GbE interfaces into a single 20Gbps LACP trunk with a static IP.

---

## 1. Prerequisites
* **Switch Configuration:** The Arista switch ports corresponding to this node must be configured as a Port Channel (LACP).
* **Cabling:** Ensure both 10GbE copper cables are connected.
* **Console Access:** Perform these steps via the local console (IPMI or physical monitor), as network connectivity may drop briefly during reconfiguration.

---

## 2. Interface Identification & Cleanup
Identify the physical device names (e.g., `eno1`, `eno2`).
```bash
nmcli device status
```

Remove any default connections created by NetworkManager during OS installation to prevent conflicts.

```bash
# Replace 'eno1' and 'eno2' with your specific device names
nmcli connection delete eno1
nmcli connection delete eno2
```

---

## 3. Configure the Bond (bond0)
We will configure an 802.3ad (LACP) bond. The lacp_rate=fast option is critical to match the Arista configuration defined in Step 1.

### A. Create the Bond Master

```Bash
nmcli connection add type bond con-name bond0 ifname bond0 \
bond.options "mode=802.3ad,miimon=100,lacp_rate=fast"
```

### B. Add Physical Slaves

Attach the physical 10GbE interfaces to the bond master.
```Bash
nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname eno1 master bond0
nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname eno2 master bond0
```

## 4. IP Configuration (Static)

The Head Node requires a static IP to function as the Cluster Controller and PXE Server.

    IP Address: 10.0.0.91

    Subnet Mask: /16 (255.255.0.0)

    Gateway: 10.0.0.254


```Bash
# Set IPv4 to Manual
nmcli connection modify bond0 ipv4.method manual

# Set the Static IP & Subnet
nmcli connection modify bond0 ipv4.addresses 10.0.0.91/16

# Set the Gateway
nmcli connection modify bond0 ipv4.gateway 10.0.0.254

# Set DNS (Google/Cloudflare used as examples, adjust if using University DNS)
nmcli connection modify bond0 ipv4.dns "8.8.8.8 1.1.1.1"
```

## 5. Activation & Verification
### A. Activate the Bond

```Bash
nmcli connection up bond0
```

### B. Verify Link Status

Check that the connection is active and green.

```Bash
nmcli connection show
```

### C. Verify LACP Negotiation (Critical)

Check the kernel bonding status to ensure the switch is cooperating.

```Bash
cat /proc/net/bonding/bond0
```

# Success Criteria:

    Bonding Mode: IEEE 802.3ad Dynamic link aggregation

    Aggregator ID: Should match on both slaves.

    Partner Mac Address: Must match the Arista Switch MAC.

        Note: If this shows 00:00:00:00:00:00, the switch is not configured correctly or cabling is faulty.

    Slave Interface state: Both should be up.