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

---

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
│ Alert & Visualization  │
└────────────────────────┘


##🎯 ObjectivesThe primary objectives of this project were to:Build an isolated SOC laboratory environment.   Deploy and configure a Wazuh Manager.   Connect a Kali Linux endpoint as a Wazuh Agent.   Validate endpoint-to-manager communication.   Generate controlled security events.   Test authentication-failure detection.   Test File Integrity Monitoring (FIM).   Generate network reconnaissance activity using Nmap.   Develop a custom Wazuh decoder.Develop a custom Wazuh detection rule.Map the detection to MITRE ATT&CK T1110 — Brute Force.Validate the detection using wazuh-logtest.Verify the alert in the Wazuh Dashboard.Build a saved search and visualization.Configure a read-only analyst role using Wazuh RBAC.

##🏗️ Lab Architecture
Network Topology Diagram
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

##Network Configuration
 MatrixEntityRoleIP AddressVMware VMnet8NAT Gateway192.168.44.2Kali LinuxAttacker / Monitored Endpoint192.168.44.130Wazuh ApplianceAll-in-One SIEM Monitoring Stack192.168.10.187

##🧰 Technology StackTechnologyPurpose🛡️ WazuhSIEM, log analysis, detection and monitoring🐉 Kali LinuxSecurity testing endpoint💻 VMware WorkstationVirtualized laboratory infrastructure🔍 Nmap / HydraNetwork reconnaissance & brute force testing📊 Wazuh DashboardAlert investigation and visualization🧪 wazuh-logtestDecoder and rule validation🎯 MITRE ATT&CKAdversary technique mapping🔐 RBACLeast-privilege SOC access control

##🔄 Log Parsing & Custom Rule Pipeline
┌───────────────────────────┐
│    /var/log/custom.log    │
│  (Raw Custom App Log)     │
└─────────────┬─────────────┘
              │ Read by Logcollector
              ▼
┌───────────────────────────┐
│  local_decoder.xml        │
│  Prematch: CUSTOMAPP      │
│  Fields: severity, user,  │
│          srcip, action    │
└─────────────┬─────────────┘
              │ Decoded Key-Value Schema
              ▼
┌───────────────────────────┐
│  local_rules.xml          │
│  Rule 100000: Base Match  │
│  Rule 100010: Fail Alert  │
│  Rule 100011: Brute Force │
└─────────────┬─────────────┘
              │ Rule Match Event
              ▼
┌───────────────────────────┐
│  alerts.json / Dashboard  │
│  Level 10 Severity Alert  │
└───────────────────────────┘
