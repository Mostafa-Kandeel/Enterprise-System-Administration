# Enterprise System Administration & Datacenter Lab

> **A production-grade, enterprise nested virtualization architecture designed as an end-to-end practical blueprint for System Administrators, Cloud Engineers, and Infrastructure Specialists.**

---

## 🖥️ Physical Host Specifications (Bare-Metal Platform)

The entire enterprise datacenter is deployed on a dedicated, high-performance bare-metal rack server engineered for heavy virtualization workloads:

| Hardware Component | Enterprise Specification | Operational Role |
| :--- | :--- | :--- |
| **Server Hardware Model** | **Dell PowerEdge R740 Rack Server** | High-performance enterprise compute platform |
| **Management IP** | `10.100.14.17` | Physical out-of-band / host management access |
| **Physical Memory (RAM)** | **74 GB** DDR4 High-Density ECC RAM | Dedicated allocation for nested hypervisors & VMs |
| **Storage Capacity** | **3 TB** High-Speed Enterprise Storage | Provisioned for nested datastores and shared pools |
| **Base Virtualization Engine** | **VMware Workstation Pro** | Hardware-Assisted Nested Virtualization (Intel VT-x/EPT) |

---

## 🏛️ vCenter Cluster & Datacenter Architecture

At the core of the infrastructure is a centralized **VMware vCenter Server Appliance (VCSA)** orchestrating a multi-host environment divided into a High-Availability Enterprise Cluster and an off-cluster Distributed Services Host:

### 🌟 High Availability & Resiliency Features (Hosts 1, 2, 3 Cluster):
* **vSphere DRS (Distributed Resource Scheduler):** Dynamically monitors CPU and RAM utilization across cluster nodes (`11.0.0.3`, `11.0.0.5`, `11.0.0.8`) and automatically balances workloads via live migration to prevent performance bottlenecks.
* **Fault Tolerance (FT) & vSphere HA:** Ensures business continuity with zero downtime. If any physical or nested host experiences failure, critical VMs fail over instantly without data loss.
* **Dedicated vMotion & Migration Network:** Optimized virtual switches and VMkernel interfaces configured specifically for seamless, zero-interruption live VM migration.
* **Centralized NAS Shared Storage:** Windows Server 2019 functions as a **NAS / Shared Storage Server**, exposing centralized shared datastores to the ESXi cluster. This enables compute-only vMotion migrations without needing to copy heavy virtual disks across hosts.

<p align="center">
  <img src="Images/Dataenter.jpg" alt="Enterprise Datacenter Overview" width="95%" />
</p>

---

## 🌐 Network Topology & Multi-Subnet Architecture

The virtual infrastructure is architected across multiple segregated subnets interconnected and protected by a central **Sophos XG Next-Generation Firewall** serving as the core router and security gateway.

```text
[ Physical Network / Internet ] (Physical Host: 10.100.14.17)
  │
  ▼ [Port 2: WAN (IP: 11.0.1.128/24)]
[ Sophos XG Firewall ] (VM on Host 11.0.0.8)
  │  Web Console: https://11.0.0.15:4444
  │  Core Router • Inter-Subnet Routing • Outbound NAT
  │
  ├──► [Port 1: LAN (IP: 11.0.0.15)] ── Subnet 11.0.0.0/24 (HA Cluster Zone)
  │     ├── ESXi Host 1 (11.0.0.3)
  │     │     ├── VMware vCenter Server Appliance (VCSA)
  │     │     └── Windows Server 2019 DC & NAS Storage
  │     ├── ESXi Host 2 (11.0.0.5)
  │     │     ├── Veeam Backup & Replication Management Server
  │     │     └── Windows 10 Enterprise Client (Domain-Joined)
  │     └── ESXi Host 3 (11.0.0.8)
  │           └── Sophos XG Firewall VM
  │
  └──► [Port 3: LAN (IP: 12.0.0.2)]  ── Subnet 12.0.0.0/24 (Distributed Services Zone)
        └── ESXi Host 4 (12.0.0.12)
              └── Dedicated Veeam Backup Proxy & Repository VM
```

### 📋 Sophos Firewall Network Interfaces & Gateway Mapping

| Interface | Zone | IP Address | Subnet Mask | Operational Function |
| :--- | :--- | :--- | :--- | :--- |
| **Port 2 (WAN)** | WAN | `11.0.1.128` | `255.255.255.0` | Uplink to Physical Network & Outbound Internet via NAT |
| **Port 1 (LAN)** | LAN | `11.0.0.15` | `255.255.255.0` | Default Gateway for Subnet `11.0.0.0/24` & Admin Web Console |
| **Port 3 (LAN)** | LAN | `12.0.0.2` | `255.255.255.0` | Default Gateway for Subnet `12.0.0.0/24` (Backup & Secondary Zone) |

<p align="center">
  <img src="Images/sophos_interfaces.png" alt="Sophos XG Network Interfaces" width="95%" />
</p>

---

## 🏢 In-Depth Host Breakdown & Workload Distribution

### 1️⃣ ESXi Host 1 (`11.0.0.3`) — Management & Enterprise Identity Core
* **Subnet:** `11.0.0.0/24` (Cluster Member)
* **Hosted Virtual Machines:**
  1. **VMware vCenter Server Appliance (VCSA):** Central management plane for the ESXi hosts, cluster features (DRS/HA/FT), virtual networking, and storage provisioning.
  2. **Windows Server 2019 (Domain Controller & Centralized Management):**
     * **Active Directory Domain Services (AD DS):** Designed to simulate a real enterprise corporate hierarchy with segregated Organizational Units (OUs), Security Groups, and Group Policy Objects (GPOs) for key departments:
       - 👥 **HR (Human Resources):** Employee accounts, departmental shares, and privacy GPOs.
       - 💻 **IT Operations:** Administrative tier accounts, management tools, and elevated rights.
       - 🎧 **Call Center / Support:** Restricted workstation policies and customer support apps.
       - 💰 **Finance & Accounting:** High-security financial resources and restricted file access.
       - 📊 **PMO (Project Management Office):** Cross-functional project access and collaboration.
     * **Core Network Services:** Windows DNS with forward/reverse lookup zones and DHCP Server.
     * **NAS / Shared Storage Server:** Configured to share centralized storage volumes via SMB/NFS/iSCSI to the ESXi cluster for live migrations.

---

### 2️⃣ ESXi Host 2 (`11.0.0.5`) — Central Data Protection & Testing
* **Subnet:** `11.0.0.0/24` (Cluster Member)
* **Hosted Virtual Machines:**
  1. **Veeam Backup & Replication Management Server:** Enterprise backup orchestration handling automated backup jobs, schedules, VM snapshots, and instant recovery.
  2. **Windows 10 Enterprise Client:** Fully domain-joined test workstation used to validate Group Policy enforcement, department user logins, permissions, and network share access.

<p align="center">
  <img src="Images/host-5.png" alt="ESXi Host 5 Dashboard" width="95%" />
</p>

---

### 3️⃣ ESXi Host 3 (`11.0.0.8`) — Perimeter Defense & Inter-Subnet Routing
* **Subnet:** `11.0.0.0/24` (Cluster Member)
* **Hosted Virtual Machines:**
  1. **Sophos XG Next-Gen Firewall Appliance:**
     * Multi-homed with 3 virtual NICs linking WAN, Subnet 11, and Subnet 12.
     * Functions as the core Layer 3 router enabling bidirectional communication between `11.0.0.0/24` and `12.0.0.0/24`.
     * Stateful packet inspection, application-level security policies, and NAT masquerading for WAN egress.

<p align="center">
  <img src="Images/host-8.png" alt="ESXi Host 8 Dashboard" width="95%" />
</p>

---

### 4️⃣ ESXi Host 4 (`12.0.0.12`) — Distributed Backup Proxy & Repository
* **Subnet:** `12.0.0.0/24` (Secondary Routed Subnet — Standalone Host)
* **Hosted Virtual Machines:**
  1. **Dedicated Veeam Backup Proxy & Backup Repository VM:**
     * Strategically placed outside the primary cluster on an independent subnet to avoid resource contention with production workloads.
     * **Load Balancing:** Offloads heavy data processing, deduplication, and compression tasks from the default backup proxy.
     * **Isolated Repository:** Provides a distributed backup storage target across network boundaries for enhanced disaster recovery readiness.

<p align="center">
  <img src="Images/host-12.png" alt="ESXi Host 12 Dashboard" width="95%" />
</p>

---

## 💾 Centralized Backup & Disaster Recovery (Veeam Engine)

The environment utilizes an enterprise-grade data protection architecture powered by **Veeam Backup & Replication (Enterprise Plus)**:

* **Active Backup Jobs:** Automated image-level, application-consistent VM backups protecting:
  - `Backup Job-Vcenter`: Full orchestration plane protection.
  - `Domain-Controller`: Active Directory system state and granular object recovery.
  - `SQL-Server`: Database-level transaction log and VM protection.
  - `Win (10) Client`: Workstation state and user data backup.
  - `Proxy`: Backup infrastructure health and state protection.
* **Performance Metrics:** Proven throughput with 2.5x deduplication/compression ratio and verified network transfer efficiency.

<p align="center">
  <img src="Images/veeam_backup_jobs.png" alt="Veeam Backup and Replication Console" width="95%" />
</p>

---

## 🗄️ Storage & Virtual Networking Infrastructure

### High-Speed Datastores
* VMFS datastores provisioned across all ESXi nodes, supplemented by the Windows Server 2019 NAS shared storage target.
* Thin and thick provisioning monitored to optimize storage capacity and VM performance.

<p align="center">
  <img src="Images/Datastore(3).jpg" alt="Datastore Management" width="95%" />
</p>

### vSphere Virtual Switches & VM Inventory
* Custom Port Groups segregating Management traffic, vMotion live migration channels, and VM Production networks.
* Complete inventory view across nested nodes.

<p align="center">
  <img src="Images/Network(3).jpg" alt="Virtual Networking" width="48%" />
  <img src="Images/Vms(3).jpg" alt="Virtual Machine Inventory" width="48%" />
</p>

---

## 🛠️ Summary of Implemented Technologies & Competencies

| Domain | Implemented Technologies & Concepts |
| :--- | :--- |
| **Virtualization Platforms** | VMware ESXi 7.x/8.x, VMware vCenter Server (VCSA), VMware Workstation Pro |
| **High Availability & Clustering** | VMware DRS (Distributed Resource Scheduler), vSphere HA, Fault Tolerance (FT), vMotion |
| **Network & Perimeter Security** | Sophos XG Firewall, Inter-Subnet / Inter-VLAN Routing, NAT Masquerading, Stateful Packet Inspection |
| **Identity & Access Management** | Microsoft Windows Server 2019 AD DS, Departmental OUs (HR, IT, Call Center, Finance, PMO), Group Policy (GPOs), DNS, DHCP |
| **Storage Architecture** | VMFS Datastores, Windows Server 2019 NAS / Shared Storage for Compute-Only vMotion Migration |
| **Backup & Disaster Recovery** | Veeam Backup & Replication (Enterprise Plus), Distributed Backup Proxies, Secondary Subnet Repository, Granular Item Recovery |

---

## 👨‍💻 Author

**Mostafa Kandeel**  
*Enterprise System Administrator & Infrastructure Engineer*  
[GitHub Profile](https://github.com/Mostafa-Kandeel)
