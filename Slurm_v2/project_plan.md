# Project Plan: Unified Nuclear Physics HPC Cluster

**Date:** December 4, 2025
**Version:** 2.1
**Target Hardware:** Supermicro "Twin" X10 (4 Nodes)
**OS Standard:** AlmaLinux 9 / OpenHPC 3.x
**Slurm Target:** Version 24.11 (Latest Stable)

---

## 1. Executive Summary
We are deploying a new dedicated HPC cluster to support low-energy nuclear physics experiments and Geant4 simulations. The system is built on a high-density 4-node Supermicro X10 server.

**Primary Goals:**
1.  **Increase Throughput:** Enable processing of significantly larger datasets than the previous infrastructure allowed.
2.  **Modernize Stack:** Upgrade from legacy Slurm 21.08.5 to the modern **Slurm 24.11+** standard.
3.  **Democratize Access:** Simplification of the user experience to reduce the training/support burden currently placed on senior graduate students (Cade).

**Key Scientific Workloads:**
* **Simulation:** Geant4, Talys (Nuclear Reaction Modeling), MCNP (if applicable).
* **Analysis:** CERN ROOT.
* **Custom DAQ Utilities:** `dat2xy`, `mvme2xy` (Data Conversion).
* **Visualization Tools:** `quick_xy_plot`, `quick_xy_rebin` (Rapid Data Inspection).

---

## 2. Hardware Architecture
The cluster will be configured as a single, high-speed homogeneous partition.

### The "Production" Partition
* **Chassis:** Supermicro 2U Twin² (4 Nodes)
* **Nodes:** 4x Supermicro X10DRT-B+
* **Processor:** 8x Xeon E5-2630 v4 (Total: 80 Cores / 160 Threads)
* **Memory:** 512GB Total (128GB per Node)
* **Interconnect:** 10GbE / InfiniBand (depending on backplane config)
* **Boot Method:** Stateless (PXE Boot / RAM Disk) via **Warewulf 4**.

---

## 3. Software Stack & User Experience

### Cluster Management: OpenHPC 3.x
* **OS:** **AlmaLinux 9**. Chosen for long-term support (2032) and binary compatibility with CERN/Fermilab standards.
* **Scheduler:** **Slurm 24.11**. This version introduces improved scheduling logic for high-throughput (many small jobs) workloads, which suits `dat2xy` data conversion tasks.

### The "Cade Relief" Strategy (User Experience)
To prevent the lead graduate student from becoming permanent IT support, we will implement **Interface Abstraction**:

1.  **Open OnDemand (Recommended Add-on):**
    * A web-based portal where users can launch Jupyter Notebooks, ROOT sessions, or file managers directly in a browser.
    * *Benefit:* Eliminates the need to teach new students how to SSH, set up X11 forwarding, or configure VNC.
2.  **Standardized Job Wrappers:**
    * Instead of users writing raw Slurm scripts, we will deploy "helper commands" globally:
        * `sub_talys input.txt` (Automatically generates the SBATCH wrapper).
        * `sub_geant4 macro.mac` (Automatically requests 20 cores).
3.  **Containerized Environments (Apptainer):**
    * We will build a single "Lab Standard" container image (`lab-physics-v1.sif`) containing ROOT, Geant4, and the custom `mvme` tools pre-compiled.
    * *Benefit:* "It works on my machine" becomes "It works on the cluster."

---

## 4. Implementation Phases

### Phase 1: The Core Build (Week 1)
* **Lead:** System Architect / Engineer
* **Tasks:**
    * Rack and stack Supermicro X10.
    * Configure Endian Firewall (DHCP Option 66/67).
    * Install AlmaLinux 9 Head Node.
    * Deploy **Warewulf 4** and boot all 4 nodes stateless.

### Phase 2: The "Scientific" Stack (Week 2)
* **Lead:** Scientific Lead (Cade)
* **Tasks:**
    * **Containerize:** Create the Apptainer definition file for the custom codes (`mvme2xy`, `dat2xy`).
    * **Compile:** Ensure Talys and Geant4 are compiled with optimizations for the Broadwell (v4) CPUs.
    * **Validation:** Verify `quick_xy_plot` works over X11 forwarding or the Web Interface.

### Phase 3: User Onboarding (Week 3)
* **Lead:** Joint Effort
* **Tasks:**
    * Deploy the "Helper Scripts" to `/usr/local/bin`.
    * Wiki Documentation: "Quickstart Guide" (max 1 page).
    * **The "Cade Rule":** If a user has a question, they check the Wiki first. If the Wiki is missing it, Cade answers it *once* and updates the Wiki.

---

## 5. Roles & Responsibilities

| Role | Responsibility | Success Metric |
| :--- | :--- | :--- |
| **System Architect** | Hardware stability, Slurm configuration, Warewulf images. | 99.9% Uptime; All nodes boot automatically. |
| **Infrastructure Eng.** | Network/Power, Endian Firewall integration. | Seamless DHCP/PXE traffic. |
| **Scientific Lead (Cade)** | Application containerization, User workflow design. | **Reduction in time spent troubleshooting peers' jobs.** |

---

## 6. Risk Assessment

1.  **Custom Code Portability:** `mvme2xy` or `dat2xy` may depend on 32-bit libraries or specific older compilers.
    * *Mitigation:* These will be permanently frozen inside an Apptainer container running an older OS (like Rocky 8) if they cannot be recompiled for AlmaLinux 9.
2.  **Storage I/O Bottlenecks:** Processing large datasets with 80 cores simultaneously may choke the NFS head node.
    * *Mitigation:* Configure a dedicated high-speed "scratch" directory on the local SSDs of the compute nodes for intermediate files.