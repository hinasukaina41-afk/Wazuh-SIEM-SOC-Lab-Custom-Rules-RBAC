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

The lab progresses from building the virtual security environment to developing and validating a custom detection pipeline.
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

---

## 🎯 Objectives

The primary objectives of this project were to:

- Build an isolated SOC laboratory environment.
- Deploy and configure a Wazuh Manager.
- Connect a Kali Linux endpoint as a Wazuh Agent.
- Validate endpoint-to-manager communication.
- Generate controlled security events.
- Test authentication-failure detection.
- Test File Integrity Monitoring (FIM).
- Generate network reconnaissance activity using Nmap.
- Develop a custom Wazuh decoder.
- Develop a custom Wazuh detection rule.
- Map the detection to **MITRE ATT&CK T1110 — Brute Force**.
- Validate the detection using `wazuh-logtest`.
- Verify the alert in the Wazuh Dashboard.
- Build a saved search and visualization.
- Configure a read-only analyst role using Wazuh RBAC.

---

## 🏗️ Lab Architecture

### Network Configuration

- **VMware VMnet8 (NAT Gateway):** `192.168.44.2`
- **Kali Linux (Attacker / Monitored Endpoint):** `192.168.44.130`
- **Wazuh Appliance (SIEM Stack):** `192.168.10.187`

---

## 🧰 Technology Stack

- 🛡️ **Wazuh:** SIEM, log analysis, detection and monitoring
- 🐉 **Kali Linux:** Security testing endpoint
- 💻 **VMware Workstation:** Virtualized laboratory infrastructure
- 🔍 **Nmap / Hydra:** Network reconnaissance & brute force testing
- 📊 **Wazuh Dashboard:** Alert investigation and visualization
- 🧪 **wazuh-logtest:** Decoder and rule validation
- 🎯 **MITRE ATT&CK:** Adversary technique mapping
- 🔐 **RBAC:** Least-privilege SOC access control

---

## ⚡ Phase 1 — Virtual SOC Lab Setup

### 1. Virtualization & Infrastructure Deployment
The lab environment was created on VMware Workstation Pro using an isolated NAT subnet (`VMnet8`).

### 2. Wazuh Deployment
The Wazuh All-in-One OVA was imported and configured with required hardware resources (4 vCPU, 8 GB RAM).

The three core Wazuh services were verified as active:
- `sudo systemctl status wazuh-manager`
- `sudo systemctl status wazuh-indexer`
- `sudo systemctl status wazuh-dashboard`

### 3. Connectivity Verification
Bidirectional communication between the endpoint and the Wazuh manager was verified using ICMP testing:
- Ping from Kali endpoint to Wazuh server: `ping -c 4 192.168.10.187`
- Ping from Wazuh server to Kali endpoint: `ping -c 4 192.168.44.130`

---

## 📥 Phase 2 — Wazuh Agent Deployment

### 1. Agent Installation
The Wazuh Agent was installed on the Kali Linux system and configured to communicate with the Wazuh Manager:
`sudo WAZUH_MANAGER="192.168.10.187" WAZUH_AGENT_NAME="kali-endpoint" apt-get install -y wazuh-agent`
`sudo systemctl enable --now wazuh-agent`

### 2. Enrollment Verification
Agent enrollment was confirmed on the Wazuh Manager CLI:
`sudo /var/ossec/bin/agent_control -l`

---

## 🚨 Phase 3 — Out-of-the-Box Security Monitoring

To validate basic security monitoring, three security events were simulated on the endpoint:

1. **Authentication Failure (SSH Brute Force):** Simulated using Hydra (`hydra -l analyst -P /usr/share/wordlists/rockyou.txt ssh://192.168.44.130 -t 4`).
2. **File Integrity Monitoring (FIM):** Triggered by altering system files under `/etc` (`sudo touch /etc/rogue_marker.conf`).
3. **Network Reconnaissance:** Generated using Nmap port scanning (`sudo nmap -sS -T4 -p 1-1024 192.168.44.130`).

---

## 🧩 Phase 4 — Custom Log Decoder & Rule Engineering

### 1. Custom Log File Setup
Registered `/var/log/custom.log` in agent `/var/ossec/etc/ossec.conf` under syslog log format.

### 2. Custom XML Decoder Development
Defined `customapp` prematch decoder and parsed fields `severity`, `user`, `srcip`, and `action` in `/var/ossec/etc/decoders/local_decoder.xml`.

### 3. Custom Detection Rule Development
Configured detection rules in `/var/ossec/etc/rules/local_rules.xml`:
- **Rule 100000 (Level 3):** Event received
- **Rule 100010 (Level 7):** Login failure event (Mapped to MITRE T1078)
- **Rule 100011 (Level 10):** Brute force threshold - 5 failures within 60s from same `srcip` (Mapped to MITRE T1110)

### 4. Rule Validation using `wazuh-logtest`
Validated log processing flow using `sudo /var/ossec/bin/wazuh-logtest`.

---

## 📊 Phase 5 — Dashboard Engineering & RBAC Governance

### 1. Saved Search & Visualization
Filtered custom events in Discover using DQL `rule.id: 100011`. Built custom dashboard `CUSTOMAPP - Cyber Soch Analyst View`.

### 2. Read-Only RBAC Implementation
Created least-privilege role `Read-Only-Analyst` mapped to user `ro.analyst`. Verified read-only enforcement in Incognito window (View access granted, modification endpoints returned HTTP 403 Forbidden).

---

## 📑 Summary & Compliance Matrix

- **File Integrity Monitoring:** NIST SP 800-53 `SI-7` | ISO/IEC 27001 `A.8.32` | MITRE ATT&CK TA0005 — Defense Evasion
- **SSH Brute Force:** NIST SP 800-53 `AU-6` | ISO/IEC 27001 `A.8.15` | MITRE ATT&CK Credential Access (T1110)
- **Nmap Scan Detection:** NIST SP 800-53 `CA-7` | ISO/IEC 27001 `A.8.16` | MITRE ATT&CK Reconnaissance (T1046)
- **Custom Rules Engine:** NIST SP 800-53 `AU-12` | ISO/IEC 27001 `A.12.4` | Detection Engineering
- **RBAC Governance:** NIST SP 800-53 `AC-2 / AC-3` | ISO/IEC 27001 `A.5.15` | Security Governance

---

## 👤 Author Information

**Hina Sukaina**  
*Cyber Security Analyst & Threat Hunter*  
- **Team:** Cyber Soch Blue Team  
- **Domain:** SOC Security Engineering
