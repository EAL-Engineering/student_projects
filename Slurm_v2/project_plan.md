# Project Plan: Unified Nuclear Physics HPC Cluster

**Date:** December 4, 2025
**Version:** 1.0
**Target Hardware:** Supermicro "Twin" X10 (Primary), Supermicro X8 (Legacy)
**OS Standard:** AlmaLinux 9 / OpenHPC 3.x

---

## 1. Executive Summary
We are deploying a new dedicated HPC cluster to support low-energy nuclear physics experiments and Geant4 simulations. The core of the system is a newly acquired 4-node Supermicro "Twin" server. We will also integrate older legacy nodes to maximize throughput.

The goal is to move from "running simulations on individual desktops" to a **centralized, queued batch system** using industry-standard tools (Slurm, OpenHPC, Apptainer) that align with major labs like CERN and Fermilab.

---

## 2. Hardware Architecture
We will organize the hardware into two Slurm partitions (queues) to ensure urgent jobs are not blocked by slower legacy hardware.

### Tier 1: The "Production" Partition (New Hardware)
* **Chassis:** Supermicro 2U Twin² (4 Nodes)
* **Nodes:** 4x Supermicro X10DRT-B+
* **Specs per Node:** Dual Xeon E5-2630 v4 (20 cores/40 threads), 128GB RAM (512GB Total Cluster RAM)
* **Role:** High-speed, fast-turnaround simulations.
* **Boot Method:** Stateless (PXE Boot / RAM Disk) via Warewulf.

### Tier 2: The "Scavenger" Partition (Legacy Hardware)
* **Nodes:** Supermicro X8DTG-QF+
* **Specs:** Dual Xeon E5630 (Westmere), 48GB RAM.
* **Role:** Long-running, low-priority jobs; testing; student learning.
* **Constraint:** These CPUs are limited to the `x86-64-v2` instruction set (No AVX).

---

## 3. Software Stack Strategy

### Operating System: AlmaLinux 9
* **Why:** Chosen to align with Fermilab/CERN standards.
* **Critical Feature:** Unlike RHEL 10, AlmaLinux 9 maintains full support for our legacy Westmere (Tier 2) CPUs.
* **Lifecycle:** Supported until 2032.

### Cluster Management: OpenHPC 3.x
* **Provisioning:** **Warewulf 4**. We will maintain a single OS image for the entire cluster.
* **Scheduler:** **Slurm**. Users will submit job scripts (`#SBATCH`) rather than running interactively.
* **Networking:** Existing Endian Firewall will handle DHCP; Head Node will handle TFTP/PXE.

### User Environment: Apptainer (Singularity)
* **Strategy:** Containerize the simulation environment to avoid "dependency hell" with different Geant4/ROOT versions.
* **Benefit:** A student can build a container on their laptop, upload it to the cluster, and it is guaranteed to run.

---

## 4. Implementation Phases

### Phase 1: Infrastructure & Head Node (Week 1)
* **Lead:** System Architect / Engineer
* **Tasks:**
    * Rack and stack the Supermicro X10.
    * Install AlmaLinux 9 manually on Node 01 (Head Node).
    * Configure Endian Firewall DHCP `next-server` option to point to Node 01.
    * Install OpenHPC base packages and Warewulf.

### Phase 2: The Stateless Fabric (Week 2)
* **Lead:** System Architect / Engineer
* **Tasks:**
    * Build the master VNFS (Virtual Node File System) image.
    * Configure Warewulf "overlays" for network (IPMI/Ethernet) configuration.
    * Boot Nodes 02, 03, and 04 over the network.
    * **Milestone:** All 4 nodes visible in `wwctl node list` and `sinfo`.

### Phase 3: Legacy Integration (Week 3)
* **Lead:** System Architect / Engineer
* **Tasks:**
    * Network boot the X8 nodes.
    * *Validation:* Verify the AlmaLinux 9 image boots on Westmere CPUs.
    * Configure Slurm partitions: `partition=main` (Default, X10 nodes) and `partition=legacy` (X8 nodes).

### Phase 4: Scientific User Pilot (Week 4)
* **Lead:** Scientific Lead (Grad Student)
* **Tasks:**
    * **Containerize Geant4:** Build an Apptainer definition file for the current simulation code.
    * **Benchmarking:** Run the standard "TestEm" Geant4 example on Tier 1 vs. Tier 2 to establish a performance baseline.
    * **Documentation:** Write a "How to Submit a Job" wiki page for the group.

---

## 5. Roles & Responsibilities

| Role | Primary Responsibility | Key Deliverable |
| :--- | :--- | :--- |
| **System Architect** | Hardware deployment, OS architecture, OpenHPC configuration. | A running Slurm cluster with 4 active nodes. |
| **Infrastructure Eng.** | Network integration (Endian), Cabling, Power/Cooling management. | DHCP option 66/67 setup, IPMI network isolation. |
| **Scientific Lead** | Software validation, Geant4 containerization, User documentation. | A working `submit_job.sh` script example for the group. |

---

## 6. Risk Assessment

1.  **Legacy CPU Instruction Sets:** The X8 nodes lack AVX instructions.
    * *Mitigation:* Compile Geant4 with `-march=generic` or `x86-64-v2` flags inside the container.
2.  **Shared Point of Failure:** The Supermicro Twin system shares a power supply backplane.
    * *Mitigation:* Ensure PSUs are connected to separate UPS circuits.
3.  **Storage:** The Head Node handles storage (NFS).
    * *Future Upgrade:* If data exceeds ~4TB, add dedicated NAS/ZFS storage.