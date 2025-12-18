# Project Plan: Unified Nuclear Physics HPC Cluster

* **Date:** December 18, 2025
* **Version:** 3.4
* **Target Hardware:** Supermicro "Twin" X10 (4 Nodes)
* **Interconnect:** 10GbE Copper with LACP Bonding
* **OS Standard:** AlmaLinux 9 / OpenHPC 3.x
* **Slurm Target:** Version 24.11

---

## 1. Executive Summary

We are deploying a new dedicated HPC cluster to support low-energy nuclear physics experiments and Geant4 simulations. The system is built on a high-density 4-node Supermicro X10 server connected via a high-speed Arista 10GbE backbone.

**Primary Goals:**

1. **Increase Throughput:** Enable processing of significantly larger datasets using high-bandwidth networked storage and parallel processing.
2. **Modernize Stack:** Upgrade from legacy Slurm 21.08.5 to the modern **Slurm 24.11+** standard.
3. **Democratize Access:** Simplification of the user experience to reduce the training/support burden currently placed on graduate students (Cade).

**Key Scientific Workloads:**

* **Simulation:** Geant4, Talys (Nuclear Reaction Modeling).
* **Analysis:** CERN ROOT.
* **Custom DAQ Utilities:** `dat2xy`, `mvme2xy` (Data Conversion).
* **Visualization Tools:** `quick_xy_plot`, `quick_xy_rebin`.

---

## 2. Hardware Architecture & Network Topology

### The "Head Node" (Node 01)

* **Role:** Cluster Controller, Storage Server, and Image Host.
* **State:** **Stateful** (OS installed on local disk).
* **Function:** Serves the OS images to the compute nodes over the local 10GbE switch to ensure rapid boot times, bypassing the external 1GbE network.

### The "Compute Nodes" (Nodes 02-04)

* **Role:** Dedicated number crunching.
* **State:** **Stateless** (Boots into RAM via PXE from Node 01).
* **Specs per Node:** 2x Xeon E5-2630 v4 (20 cores), 128GB RAM.

### Network Fabric

* **Switch:** Arista 10GbE Copper Switch.
* **Configuration:** LACP Link Bonding (802.3ad).
* **Bandwidth:** 20 Gbps aggregated bandwidth per node.
* **Boot Flow:** 1. DHCP Request (via External Endian Firewall).
    2. TFTP/HTTP Image Transfer (Local 10GbE transfer from Node 01).

---

## 3. Software Stack & User Experience

### Cluster Management: OpenHPC 3.x

* **OS:** **AlmaLinux 9** (Binary compatible with CERN/Fermilab standards).
* **Scheduler:** **Slurm 24.11**.
* **Provisioning:** **Warewulf 4**. Configured to handle the handoff from single-NIC PXE boot to bonded-NIC OS runtime.

### The "Cade Relief" Strategy (User Experience)

To prevent the lead graduate student from becoming permanent IT support, we will implement **Interface Abstraction**:

1. **Open OnDemand:** A web-based portal for Jupyter Notebooks, file management, and shell access.
2. **Standardized Job Wrappers:**
    * Global helper commands (e.g., `sub_talys`, `sub_geant4`) to auto-generate SBATCH headers.
3. **Containerized Environments (Apptainer):**
    * A "Lab Standard" container image (`lab-physics-v1.sif`) containing ROOT, Geant4, and custom `mvme` tools.

### Architecture Decision: Apptainer vs. Integrated Image

**Question:** Why use external containers instead of installing scientific software directly into the boot image?

**Decision:** We will host the software stack in Apptainer containers on shared storage, rather than "baking" it into the Warewulf VNFS image.

**Rationale:**

1. **Boot Speed:** Keeps the VNFS OS image small (<500MB) for rapid PXE booting. Adding Geant4/ROOT would balloon the image to 4GB+, significantly slowing down node reboots.
2. **Maintainability:** Allows us to update the scientific software (e.g., a new Geant4 version) instantly by replacing the `.sif` file, without needing to rebuild the OS image or reboot the entire cluster.
3. **Performance:** The **20Gbps LACP interconnect** ensures that executing the container from the network storage has near-local performance, negating the traditional speed penalty of network shares.

---

## 4. Implementation Phase Strategy

### Phase 1: Infrastructure & Networking

* **Lead:** Greg & Don
* **Tasks:**
  1. Configure Arista Switch: Set up Port Channels (LACP) for all node ports.
  1. Configure Endian Firewall: Point DHCP Option 66 (Next-Server) to the IP of Node 01.
  1. Install AlmaLinux 9 on Node 01 (Head Node).
  1. Configure Network Bonding (`bond0`) on the Head Node.

### Phase 2: The Stateless Fabric

* **Lead:** Greg & Don
* **Tasks:**
  1. Install Warewulf 4 on the Head Node.
  1. Build the master VNFS image (AlmaLinux 9 base).
  1. Configure Warewulf Overlays to inject `ifcfg-bond0` configurations into compute nodes during boot.
  1. Boot Nodes 02-04 and verify LACP negotiation on the Arista switch.
  1. Configure NFS Server on the Head Node.
  1. Establish central software repository (`/home/software`).
  1. Create `nfs-client` system overlay to auto-mount storage on compute nodes.

### Phase 3: Scientific Stack & Validation

* **Lead:** Cade
* **Tasks:**
  1. **Containerize:** Create Apptainer definitions for `mvme2xy`, `dat2xy`, Geant4, etc.
  1. **Wrappers:** Create `sub_talys`, `sub_geant4` helper scripts.
  1. **Validation:** Verify 20Gbps throughput using `iperf3` and run "Golden Ticket" simulation job.

### Phase 4: User Onboarding

* **Lead:** Joint Effort
* **Tasks:**
  1. Publish "Quickstart Guide" on internal Wiki.
  1. Train initial user group.

---

## 5. Roles & Responsibilities

| Role | Responsibility | Success Metric |
| :--- | :--- | :--- |
| **Greg & Don** | Hardware integration, Slurm/Warewulf config, Network Bonding, Endian Firewall integration. | Nodes boot via PXE; 20Gbps bandwidth active; Seamless DHCP redirection. |
| **Cade** | Application containerization, User workflow design, Web Portal testing. | Reduction in time spent troubleshooting peers' jobs. |

---

## 6. Risk Assessment

1. **LACP/PXE Handoff:** PXE requires a single interface, while the OS wants a bond.
    * *Mitigation:* Warewulf configuration must specifically delay the bonding initialization until after the image is transferred, or use Mode 4 (802.3ad) compatible boot settings.
2. **Storage I/O:** Processing large datasets with 80 cores simultaneously may choke the NFS head node.
    * *Mitigation:* Use the Head Node's storage primarily for `/home`; use local node SSD scratch space for heavy write operations.
