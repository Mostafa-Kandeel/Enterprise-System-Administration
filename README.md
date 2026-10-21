# Enterprise-System-Administration

A comprehensive enterprise-grade nested virtualization home lab built using **VMware ESXi**, **vCenter Server**, **Microsoft Windows Server**, **Sophos XG Firewall**, and **Veeam Backup & Replication**.

---

## 📌 Project Overview

This project demonstrates the design, deployment, configuration, and administration of a full enterprise IT infrastructure in a nested virtualization environment. It covers core enterprise system administration pillars, including:

* **Compute & Virtualization:** Nested VMware ESXi cluster managed by VMware vCenter Server.
* **Network & Security:** Perimeter firewall and routing configured via Sophos Firewall, segmenting traffic across isolated VLANs and subnets.
* **Identity & Directory Services:** Windows Server Active Directory Domain Services (AD DS), DNS, and DHCP.
* **High Availability & Storage:** VMFS datastores, vSphere networking (Standard/Distributed Switches), and workload balance.
* **Data Protection & Disaster Recovery:** Enterprise backup and restore strategies with Veeam Backup & Replication.

---

## 🏗️ Architecture & Topology

![Datacenter Overview](Images/Dataenter.jpg)

### Network Segmentation & VLANs

| Zone / VLAN | Subnet | Description | Gateway |
| :--- | :--- | :--- | :--- |
| **Management** | `192.168.1.0/24` | ESXi hosts, vCenter Server Appliance (VCSA) | `192.168.1.1` |
| **Corporate / LAN** | `10.10.10.0/24` | Domain Controllers, Member Servers, Workstations | `10.10.10.1` |
| **Storage / vMotion** | `10.10.20.0/24` | VM migration & storage traffic | `10.10.20.1` |
| **DMZ / Perimeter** | `172.16.1.0/24` | Public-facing services & reverse proxies | `172.16.1.1` |

---

## ⚙️ Core Infrastructure Components

### 1. Virtualization Layer (VMware vSphere)
* **vCenter Server**: Centralized cluster management, DRS, and resource monitoring.
* **Nested ESXi Hosts**: Multiple ESXi hypervisors deployed to simulate enterprise datacenter conditions.

![Host Overview](Images/host-5.jpg)

### 2. Virtual Networking
* Configured virtual switches (vSwitches / DVS) with dedicated port groups for VM traffic, management, and storage.
* Network traffic isolation and security policies.

![Virtual Networking](Images/Network(3).jpg)

### 3. Shared Storage & Datastores
* Provisioned VMFS datastores with tiered storage architecture.
* Monitored thin/thick provisioning and storage allocation.

![Datastore Overview](Images/Datastore(3).jpg)

### 4. Virtual Machines & Workloads
* Deployment of enterprise services including Domain Controllers, management servers, and testing workstations.

![Virtual Machines Inventory](Images/Vms(3).jpg)

---

## 🛡️ Identity, Security & Network Services

### Microsoft Windows Server & Active Directory (AD DS)
* **Domain Services:** Forest and domain deployment, OU hierarchy design, and security group nesting.
* **Group Policy Management (GPO):** Enforcing password policies, firewall baselines, and desktop restrictions.
* **Core Network Services:** Integrated forward/reverse DNS zones and redundant DHCP scopes with reservations.

### Sophos Firewall & Perimeter Security
* Stateful packet inspection and Layer 7 application filtering.
* Inter-VLAN routing, NAT rules, and firewall rules governing ingress/egress traffic.
* Threat prevention and administrative VPN access.

---

## 💾 Backup & Disaster Recovery (Veeam)
* **Backup Repository Configuration:** Isolated backup storage target.
* **Backup Jobs:** Consistent VM-level image backups leveraging VMware VADP (vSphere API for Data Protection).
* **Recovery Testing:** Verified instant VM recovery, guest file restoration, and application-item recovery (Active Directory objects).

---

## 📊 Summary of Implemented Technologies

| Category | Technology |
| :--- | :--- |
| **Hypervisor** | VMware ESXi 7.x / 8.x |
| **Central Management** | VMware vCenter Server Appliance (VCSA) |
| **Directory & Network** | Windows Server (Active Directory, DNS, DHCP, GPO) |
| **Firewall & Routing** | Sophos XG Firewall |
| **Backup & DR** | Veeam Backup & Replication |

---

## 👨‍💻 Author

**Mostafa Kandeel**  
*Enterprise System & Infrastructure Enthusiast*  
[GitHub Profile](https://github.com/Mostafa-Kandeel)
