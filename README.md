# 🛡️ Wazuh SOC Detection & Monitoring Lab

[![SOC](https://img.shields.io/badge/SOC-Blue%20Team-blue)](https://wazuh.com/)
[![DETECTION LAB](https://img.shields.io/badge/DETECTION%20LAB-Wazuh-brightgreen)](https://wazuh.com/)
[![SIEM](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blue)](https://wazuh.com/)
[![KALI LINUX](https://img.shields.io/badge/KALI%20LINUX-Attacker-red)](https://www.kali.org/)
[![VMWARE](https://img.shields.io/badge/VMware-NAT%20Subnet-orange)](https://www.vmware.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110-pink)](https://attack.mitre.org/)
[![Detection](https://img.shields.io/badge/Detection-Validated-green)](https://wazuh.com/)
[![RBAC](https://img.shields.io/badge/RBAC-Read--Only-lightblue)](https://wazuh.com/)

🔎 **Practical Security Monitoring • Custom Detection Engineering • SOC Operations**

## 🚀 Project Overview

This project documents the development of a **hands-on Security Operations Center (SOC) monitoring lab** using **Wazuh SIEM, Kali Linux, and VMware Workstation Pro**.

The lab progresses from building the virtual security environment to developing and validating a custom detection pipeline:

```text
┌────────────────────────┐
│     SECURITY EVENT     │
│  Failed Login Attempt  │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│      WAZUH AGENT       │
│ Logs collected & sent  │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│     WAZUH MANAGER      │
│ Decoded & Rule Matched │
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│    WAZUH DASHBOARD     │
│ Alert → Investigation →│
│ Visualization → Response│
└────────────────────────┘
```
---
```

## 🎯 Objectives
The primary objectives of this project were to:

Build an isolated SOC laboratory environment.

Deploy and configure a Wazuh Manager.

Connect a Kali Linux endpoint as a Wazuh Agent.

Validate endpoint-to-manager communication.

Generate controlled security events.

Test authentication-failure detection.

Test File Integrity Monitoring (FIM).

Generate network reconnaissance activity using Nmap.

Develop a custom Wazuh decoder.

Develop a custom Wazuh detection rule.

Map the detection to MITRE ATT&CK T1110 — Brute Force.

Validate the detection using wazuh-logtest.

Verify the alert in the Wazuh Dashboard.

Build a saved search and visualization.

Configure a read-only analyst role using Wazuh RBAC.
```
---

## 🏗️ Lab Architecture
```text

┌─────────────────────────────────────────┐
│               HOST SYSTEM               │
│                 Windows                 │
└────────────────────┬────────────────────┘
                     │
         VMware Workstation NAT Network
                192.168.44.0/24
                     │
        ┌────────────┴────────────┐
        │                         │
┌───────▼────────┐        ┌───────▼────────┐
│   Kali Linux   │        │  Wazuh Server  │
│                │        │                │
│ 192.168.44.130 │        │ 192.168.10.187 │
│                │        │                │
│  Wazuh Agent   ├───────►│  Wazuh Manager │
│  Nmap / Hydra  │  Logs  │  Wazuh Indexer │
│  Test Events   │        │ Wazuh Dashboard│
└────────────────┘        └────────────────┘
```
---
## Network Configuration Matrix
| Component | Configuration |
| :--- | :--- |
| Network | VMware NAT Network (VMnet8) |
| Subnet | `192.168.xx.xx/xx` |
| Gateway (NAT) | `192.168.xx.xx` |
| DHCP Scope | `192.168.xx.xx - 192.168.xx.xx` |
| Wazuh Server (SIEM) | `192.168.xx.xx` |
| Kali Linux (Attacker) | `192.168.xx.xx` |
| Monitoring Platform | Wazuh (All-in-One) |
```
---
## 💼 Technology Stack
```
| Technology | Purpose |
| :--- | :--- |
| 🛡️ **Wazuh** | SIEM, XDR, log analysis, detection and monitoring[cite: 14] |
| 🐉 **Kali Linux** | Offensive security testing endpoint[cite: 14] |
| 💻 **VMware Workstation Pro** | Virtualized laboratory infrastructure (Type 2 Hypervisor)[cite: 14] |
| 🔍 **Nmap** | Network reconnaissance testing[cite: 14] |
| 🗡️ **Hydra** | SSH brute-force simulation[cite: 14] |
| 📊 **Wazuh Dashboard** | Alert investigation and visualization[cite: 14] |
| 🧪 **wazuh-logtest** | Decoder and rule validation[cite: 15] |
| 🎯 **MITRE ATT&CK** | Adversary technique mapping[cite: 15] |
| 🔐 **RBAC** | Least-privilege SOC access control[cite: 15] |
```
---

## 🚀 Phase 1 — Virtual SOC Lab Setup & Monitoring Fundamentals[cite: 15, 16]

### 1. VMware Workstation Setup & Network Architecture[cite: 16]

The laboratory is hosted on **VMware Workstation Pro**. All guests are attached to a NAT-backed virtual network (VMnet8). The NAT subnet is configured with a gateway at `.2` and a DHCP scope supplying dynamic leases. Static reservations were applied to the monitoring host to guarantee address stability for agent enrolment[cite: 16].

**Rationale for selecting NAT:**[cite: 16]

- **Isolation:** Guests are not exposed on the physical LAN, containing attack traffic[cite: 16].
- **Controlled Internet Access:** Outbound connectivity is preserved through host address translation[cite: 16].
- **Host Communication:** The VMnet8 host adapter enables the analyst to reach the Wazuh dashboard over HTTPS[cite: 17].
- **Deterministic Addressing:** A private, hypervisor-managed DHCP scope avoids collisions[cite: 17].

---

### 2. Kali Linux & Wazuh OVA Import & Boot Verification[cite: 17]

Both appliances were deployed from OVA images. The import procedure involved:[cite: 17]

- Selecting `File > Open` in VMware Workstation and choosing the downloaded `.ova` file[cite: 17].
- Accepting the OVF licence agreement and selecting a storage path[cite: 17].
- Allocating resources: **Wazuh** (4 vCPU / 8 GB RAM); **Kali** (2 vCPU / 4 GB RAM)[cite: 18].
- Setting the network adapter of each VM to **NAT (VMnet8)**[cite: 18].
- Powering on each VM and recording the leased IP address[cite: 18].

**Boot Verification Commands:**[cite: 18]

```bash
# On Kali
ip -brief address show
hostnamectl
uptime

# On Wazuh
ip -brief address show
hostnamectl
uptime

