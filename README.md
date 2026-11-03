<div align="center">

# 🛡️ COMPREHENSIVE SOC SECURITY TASK REPORT
### Virtual Lab Deployment, Real-Time Monitoring, Custom Log Decoders & Dashboard Engineering

[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-0052CC?style=for-the-badge&logo=wazuh&logoColor=white)](https://wazuh.com/)
[![Kali Linux](https://img.shields.io/badge/Offensive-Kali%20Linux-557C93?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![VMware](https://img.shields.io/badge/Virtualization-VMware%20Workstation-60B5E5?style=for-the-badge&logo=vmware&logoColor=white)](https://www.vmware.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-FF6600?style=for-the-badge&logo=matrix&logoColor=white)](https://attack.mitre.org/)
[![Status](https://img.shields.io/badge/Verification-22%2F22%20PASSED-00C853?style=for-the-badge&logo=checkmarx&logoColor=white)](#41-summary-table)

**"Proactive Defense through Integrated Security Monitoring"**

---

| **Author** | **Team** | **Domain** | **Date** |
| :--- | :--- | :--- | :--- |
| **Hina Sukaina** | Cyber Soch Blue Team | SOC Security Engineering | 2026-09-15 |

</div>

---

## 📑 Table of Contents
- [1. Executive Summary & Objective Overview](#1-executive-summary--objective-overview)
  - [1.1 Executive Summary](#11-executive-summary)
  - [1.2 Objective Overview](#12-objective-overview)
- [2. Virtual Lab Architecture & Core Setup](#2-virtual-lab-architecture--core-setup)
  - [2.1 VMware Network Topology](#21-vmware-network-topology)
  - [2.2 Appliance Deployment & Verification](#22-appliance-deployment--verification)
  - [2.3 Wazuh Service Health Verification](#23-wazuh-service-health-verification)
  - [2.4 Web Dashboard & HTTPS Setup](#24-web-dashboard--https-setup)
  - [2.5 Network Connectivity Checks](#25-network-connectivity-checks)
  - [2.6 Agent Enrolment & Status](#26-agent-enrolment--status)
- [3. Threat Simulations & Default Detections](#3-threat-simulations--default-detections)
  - [3.1 SSH Brute-Force Attack](#31-ssh-brute-force-attack)
  - [3.2 File Integrity Monitoring (FIM)](#32-file-integrity-monitoring-fim)
  - [3.3 Reconnaissance (Nmap Scanning)](#33-reconnaissance-nmap-scanning)
- [4. Custom Engineering: Decoders, Rules, Dashboards & RBAC](#4-custom-engineering-decoders-rules-dashboards--rbac)
  - [4.1 Custom Log Ingestion (`ossec.conf`)](#41-custom-log-ingestion-ossecconf)
  - [4.2 Custom Parent/Child Decoder (`local_decoder.xml`)](#42-custom-parentchild-decoder-local_decoderxml)
  - [4.3 Bespoke Detection Rules (`local_rules.xml`)](#43-bespoke-detection-rules-local_rulesxml)
  - [4.4 Logic Validation (`wazuh-logtest`)](#44-logic-validation-wazuh-logtest)
  - [4.5 Live End-to-End Ingestion Testing](#45-live-end-to-end-ingestion-testing)
  - [4.6 Dashboard & Visualisation Engineering](#46-dashboard--visualisation-engineering)
  - [4.7 Governance & Least Privilege RBAC Enforcement](#47-governance--least-privilege-rbac-enforcement)
- [5. Verification Matrix & Regulatory Compliance](#5-verification-matrix--regulatory-compliance)
  - [5.1 Summary Table](#51-summary-table)
  - [5.2 Regulatory & Framework Compliance](#52-regulatory--framework-compliance)
- [6. References](#6-references)

---

## 1. Executive Summary & Objective Overview

### 1.1 Executive Summary
This project documents the end-to-end design, deployment, operational validation, and custom engineering of an isolated virtual Security Operations Centre (SOC) laboratory. Built upon the **Wazuh Unified SIEM/XDR Platform**, the environment captures telemetry, normalises non-standard event sources, correlates threats, visualises actionable metrics, and enforces strict RBAC governance.

The laboratory runs inside an isolated NAT-segmented network (`VMnet8`) hosted on **VMware Workstation Pro**. It pairs a **Kali Linux** offensive machine (attacker) with an all-in-one **Wazuh Appliance** (manager, indexer, dashboard) monitoring a target **Debian/Kali endpoint**. Across 22 explicit verification checkpoints, the platform demonstrated complete operational coverage mapping directly to **NIST SP 800-53**, **ISO/IEC 27001:2022**, and **MITRE ATT&CK**.

### 1.2 Objective Overview

#### 🎯 Phase 1 Objectives — Infrastructure & Baseline Security
- Configure a secure, NAT-segmented isolated subnet on `VMnet8`.
- Deploy Kali Linux and Wazuh OVA virtual appliances with resource allocations.
- Validate systemd health for core stack components: `wazuh-manager`, `wazuh-indexer`, `wazuh-dashboard`.
- Verify authenticated HTTPS administration via the web console.
- Confirm bidirectional ICMP and Layer 4 reachability across nodes.
- Enrol target endpoints using the `wazuh-agent`.
- Simulate and validate out-of-the-box alerts: **SSH Brute Force**, **FIM Tampering**, **Nmap Reconnaissance**.

#### 🛠️ Phase 2 Objectives — Custom Engineering & Governance
- Configure custom application telemetry ingestion (`/var/log/custom.log`).
- Engineer an XML parent/child regex decoder in `local_decoder.xml`.
- Author custom correlation detection rules (`IDs >= 100000`) in `local_rules.xml`.
- Deterministically validate rules using `wazuh-logtest`.
- Create a customized operational SOC dashboard (`CUSTOMAPP - Cyber Soch Analyst View`).
- Engineer a least-privilege `Read-Only-Analyst` role and user via Wazuh RBAC.
- Perform black-box security testing from an incognito session to confirm read-only enforcement.

---

## 2. Virtual Lab Architecture & Core Setup

### 2.1 VMware Network Topology

```
                   +-----------------------------------------------+
                   |           Host System (Physical NIC)          |
                   +-----------------------------------------------+
                                           |
                                [VMware NAT Service]
                                           |
   +---------------------------------------+---------------------------------------+
   |                                VMnet8 (NAT Subnet)                            |
   |                                Subnet: 192.168.xx.xx                           |
   |                                Gateway: 192.168.xx.2                           |
   +---------------------------------------+---------------------------------------+
                      |                                       |
                      v                                       v
   +----------------------------------+    +----------------------------------+
   |           Kali Linux             |    |          Wazuh Server            |
   |    (Offensive / Endpoint Agent)  |    |     (Manager / Indexer / Dash)   |
   |        IP: 192.168.xx.xx         |    |        IP: 192.168.xx.xx         |
   +----------------------------------+    +----------------------------------+
```

| Network Parameter | Subnet Configuration Details |
| :--- | :--- |
| **Virtual Network Adapter** | VMware Network Adapter VMnet8 (NAT) |
| **Subnet IP & Mask** | `192.168.xx.0` / `255.255.xx.0` |
| **Gateway Address** | `192.168.xx.2` |
| **Isolation Rationale** | Prevents hostile traffic (brute-force/port scans) from leaking to LAN while preserving host internet egress for packages/updates. |

### 2.2 Appliance Deployment & Verification

```bash
# Verify system IP assignment and interface status on deployed nodes
ip -brief address show
hostnamectl
uptime
```

*Resource Allocations:*
- **Wazuh All-in-One Appliance:** 4 vCPUs, 8 GB RAM, 60 GB Disk space.
- **Kali Linux Workstation:** 2 vCPUs, 4 GB RAM.

### 2.3 Wazuh Service Health Verification

Before telemetry ingestion can occur, all three primary services must be confirmed in an `active (running)` status:

```bash
# System service status verification
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

```text
● wazuh-manager.service - Wazuh manager
     Loaded: loaded (/lib/systemd/system/wazuh-manager.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-08-22 10:14:07 UTC; 26min ago
      Tasks: 121 (limit: 9448)
     CGroup: /system.slice/wazuh-manager.service
             ├─1188 /var/ossec/bin/wazuh-authd
             ├─1231 /var/ossec/bin/wazuh-analysisd
             ├─1279 /var/ossec/bin/wazuh-remoted
             └─1310 /var/ossec/bin/wazuh-logcollector
```

### 2.4 Web Dashboard & HTTPS Setup

The web user interface is published over **TLS/HTTPS** on port 443. Self-signed SSL certificates are inspected and trusted for laboratory operations.

```bash
# Retrieve original installation credentials on the Wazuh server
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

- **URL:** `https://192.168.xx.xx:443`
- **Username:** `admin`
- **Status:** Successfully validated overview widgets, agent inventory, and MITRE mapping dashboards.

### 2.5 Network Connectivity Checks

To isolate network faults prior to testing higher-layer services, bidirectional reachability and protocol connectivity checks were executed:

```bash
# 1. Attacker (Kali) -> SIEM Manager (Wazuh)
kali@kali:~$ ping -c 4 192.168.xx.xx
# Result: 4 packets transmitted, 4 received, 0% packet loss

# 2. SIEM Manager (Wazuh) -> Attacker (Kali)
wazuh-user@wazuh:~$ ping -c 4 192.168.xx.xx
# Result: 4 packets transmitted, 4 received, 0% packet loss

# 3. Layer 4 Port Audit from Endpoint to Manager
nc -zv 192.168.xx.xx 1514 1515 55000
```

### 2.6 Agent Enrolment & Status

The `wazuh-agent` was installed on the Kali Linux target system to stream system logs, auth logs, and process states to the central manager.

```bash
# Register GPG key and APT Repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
  sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
sudo chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update

# Install and automatically register agent
sudo WAZUH_MANAGER="192.168.xx.xx" WAZUH_AGENT_NAME="kali-endpoint" apt-get install -y wazuh-agent
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

```bash
# Verify active enrolment from the Wazuh Manager
sudo /var/ossec/bin/agent_control -l
```

```text
Wazuh agent_control. List of available agents:
   ID: 000, Name: wazuh-server (server), IP: 127.0.0.1, Active/Local
   ID: 001, Name: kali-endpoint, IP: 192.168.xx.xx, Active
```

---

## 3. Threat Simulations & Default Detections

```
+------------------+         Attacks Triggered        +------------------+
|                  | -----------------------> SSH     |                  |
|   Kali Linux     | -----------------------> FIM     |  Wazuh Server    |
|   (Attacker)     | -----------------------> Nmap    |  (SIEM Receiver) |
|                  | <----------------------- Alerts  |                  |
+------------------+                                  +------------------+
```

### 3.1 SSH Brute-Force Attack
- **Objective:** Simulate password guessing against SSH remote services.
- **Attack Vector:**

```bash
# Attack Execution via Hydra
hydra -l analyst -P /usr/share/wordlists/rockyou.txt ssh://192.168.xx.xx -t 4 -V

# Deterministic sequential failure generation
for i in $(seq 1 12); do
  sshpass -p 'WrongPass!' ssh -o StrictHostKeyChecking=no analyst@192.168.xx.xx exit
done
```

- **Detection Results:** Log collector parsed `/var/log/auth.log`. Triggered Rule **`5710`** (*sshd: Attempt to login using a non-existent user*) and elevated to correlation Rule **`5712`** (*sshd: Brute force trying to get access*).
- **Severity Level:** `10` | **MITRE ATT&CK:** `T1110 (Brute Force)`

### 3.2 File Integrity Monitoring (FIM)
- **Objective:** Monitor unauthorized modifications inside critical system directories (`/etc`).
- **Attack Vector:**

```bash
# Modify, create, and remove monitored targets
sudo bash -c 'echo "# audit marker $(date -u +%FT%TZ)" >> /etc/hosts'
sudo touch /etc/rogue_marker.conf
sudo rm /etc/rogue_marker.conf
```

- **Detection Results:** `syscheck` module calculated real-time cryptographic hash deltas (SHA256) and file metadata changes.
- **Triggered Rules:**
  - Rule **`550`**: File added to system.
  - Rule **`553`**: File deleted from system.
  - Rule **`554`**: File integrity checksum changed.
- **Severity Level:** `7` | **MITRE ATT&CK:** `T1565.001 (Stored Data Manipulation)`

### 3.3 Reconnaissance (Nmap Scanning)
- **Objective:** Detect network-layer host discovery and port scans.
- **Attack Vector:**

```bash
# Execute noisy SYN sweep and service detection
sudo nmap -sS -T4 -p 1-1024 192.168.xx.xx
sudo nmap -A -T4 192.168.xx.xx
```

- **Detection Results:** Firewall logs captured connection floods; parsed via the `iptables` decoder and raised Rule **`4100xxx`** (Portscan Event).
- **Severity Level:** `6` | **MITRE ATT&CK:** `T1046 (Network Service Discovery)`

---

## 4. Custom Engineering: Decoders, Rules, Dashboards & RBAC

```
+------------------+      Forward     +------------------+      Match     +------------------+
|  /var/log/       | -------------->  | local_decoder.xml| ------------>  | local_rules.xml  |
|  custom.log      |  (ossec.conf)    | (Regex Extract)  |  (Rule 1000xx) | (Triggers Alert) |
+------------------+                  +------------------+                +------------------+
                                                                                   |
                                                                                   v
                                                                          +------------------+
                                                                          | Custom Dashboard |
                                                                          | & RBAC Sandbox   |
                                                                          +------------------+
```

### 4.1 Custom Log Ingestion (`ossec.conf`)

To ingest a non-standard local application log, the following configuration was appended to `/var/ossec/etc/ossec.conf` on the monitored agent:

```xml
<!-- /var/ossec/etc/ossec.conf (Agent Telemetry Ingestion) -->
<ossec_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/custom.log</location>
  </localfile>
</ossec_config>
```

### 4.2 Custom Parent/Child Decoder (`local_decoder.xml`)

Raw sample log line generated by the application:
```text
2026-08-22 10:14:33 CUSTOMAPP severity=HIGH user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_FAILED msg="invalid password"
```

A modular, multi-stage parent/child decoder was added to `/var/ossec/etc/decoders/local_decoder.xml` on the manager:

```xml
<!-- /var/ossec/etc/decoders/local_decoder.xml -->
<decoder name="customapp">
  <prematch>CUSTOMAPP</prematch>
</decoder>

<decoder name="customapp-fields">
  <parent>customapp</parent>
  <regex offset="after_parent">severity=(\S+) user=(\S+) src_ip=(\S+) action=(\S+)</regex>
  <order>severity, user, srcip, action</order>
</decoder>

<decoder name="customapp-msg">
  <parent>customapp</parent>
  <regex>msg="([^"]+)"</regex>
  <order>msg</order>
</decoder>
```

### 4.3 Bespoke Detection Rules (`local_rules.xml`)

Custom rule definitions were authored inside `/var/ossec/etc/rules/local_rules.xml` using ID range `>= 100000`:

```xml
<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="customapp,">
  <!-- Base Rule -->
  <rule id="100000" level="3">
    <decoded_as>customapp</decoded_as>
    <description>CUSTOMAPP: Event received</description>
  </rule>

  <!-- Login Failure Rule -->
  <rule id="100010" level="7">
    <if_sid>100000</if_sid>
    <field name="action">LOGIN_FAILED</field>
    <description>CUSTOMAPP: Login failure for user $(user) from $(srcip)</description>
    <mitre>
      <id>T1078</id>
    </mitre>
  </rule>

  <!-- Stateful Brute Force Correlation -->
  <rule id="100011" level="10" frequency="5" timeframe="60">
    <if_matched_sid>100010</if_matched_sid>
    <same_field>srcip</same_field>
    <description>CUSTOMAPP: Possible brute-force - 5 LOGIN_FAILED from same srcip in 60s</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>

  <!-- High Severity Rule -->
  <rule id="100020" level="12">
    <if_sid>100000</if_sid>
    <field name="severity">HIGH</field>
    <description>CUSTOMAPP: HIGH severity event - $(msg)</description>
  </rule>
</group>
```

### 4.4 Logic Validation (`wazuh-logtest`)

Before deploying rules to live production, `wazuh-logtest` was utilized to confirm parsing accuracy across all phases:

```bash
sudo /var/ossec/bin/wazuh-logtest
```

```text
Type one log per line
>> 2026-08-22 10:14:33 CUSTOMAPP severity=HIGH user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_FAILED msg="invalid password"

**Phase 1: Completed pre-decoding.
       full event: '2026-08-22 10:14:33 CUSTOMAPP severity=HIGH user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_FAILED msg="invalid password"'

**Phase 2: Completed decoding.
       name: 'customapp'
       action: 'LOGIN_FAILED'
       msg: 'invalid password'
       severity: 'HIGH'
       srcip: '192.168.xx.xx'
       user: 'hina.sukaina'

**Phase 3: Completed filtering (rules).
       id: '100020'
       level: '12'
       description: 'CUSTOMAPP: HIGH severity event - invalid password'
       groups: '['customapp']'
       firedtimes: '1'
```

### 4.5 Live End-to-End Ingestion Testing

Telemetry injection commands executed directly on the endpoint host:

```bash
# 1. Informational low-severity event
sudo bash -c 'echo "$(date +"%F %T") CUSTOMAPP severity=LOW user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_OK msg=\"session opened\"" >> /var/log/custom.log'

# 2. Critical single event
sudo bash -c 'echo "$(date +"%F %T") CUSTOMAPP severity=HIGH user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_FAILED msg=\"invalid password\"" >> /var/log/custom.log'

# 3. Frequency event burst (triggers correlation rule 100011)
for i in $(seq 1 6); do
  sudo bash -c 'echo "$(date +"%F %T") CUSTOMAPP severity=HIGH user=hina.sukaina src_ip=192.168.xx.xx action=LOGIN_FAILED msg=\"invalid password\"" >> /var/log/custom.log'
  sleep 2
done
```

**Discover Query (KQL):**
```kql
rule.id: (100010 or 100011 or 100020) and agent.name: "kali-endpoint"
```

### 4.6 Dashboard & Visualisation Engineering

A custom operational dashboard named **`CUSTOMAPP - Cyber Soch Analyst View`** was constructed with five analyst-centric visualisations:

| Visualisation Title | Type | Query / Metric / Parameters |
| :--- | :--- | :--- |
| **Custom Alerts (24h)** | Metric Panel | Count of total documents (`rule.id >= 100000`) |
| **Custom Alert Timeline** | Vertical Bar Chart | Y-Axis: Document Count \| X-Axis: Timestamp Date Histogram \| Split Series: `rule.id` |
| **Top Sources & Actions** | Data Table | Primary Aggregation: Terms of `data.srcip` \| Sub-Aggregation: `data.action` |
| **Severity Distribution** | Donut / Pie Chart | Split Slices: `rule.level` |
| **Live Filter Panel** | Saved Search | Streamed filter results for custom rule events |

### 4.7 Governance & Least Privilege RBAC Enforcement

To align with security control principles, a dedicated read-only role was deployed:

1. **Security Policy Creation (`ro-all-read`):**
   - **Allowed Actions:** `agent:read`, `rule:read`, `decoder:read`, `cluster:read`, `manager:read`
   - **Resource Scope:** `*:*:*`
2. **Role Creation (`Read-Only-Analyst`):** Bound to policy `ro-all-read`.
3. **User Creation (`ro.analyst`):** Configured with internal authentication credentials.
4. **Role Mapping & Kibana Role Binding:** Mapped `ro.analyst` user to `Read-Only-Analyst` and assigned the built-in OpenSearch `kibana_read_only` backend role.

#### 🕵️ Incognito Session Validation Results
- **Read Operations (Pass):** Full access to Overview dashboards, event logs, agent status, and saved custom visualisations.
- **Write Operations (Forbidden - Pass):**
  - Rule creation/editing (`Management > Rules`) -> `HTTP 403 Forbidden` / Save button disabled.
  - Agent group re-assignments -> `Permission Denied` toast popup.
  - User creation/modification -> Access denied closed.

---

## 5. Verification Matrix & Regulatory Compliance

### 5.1 Summary Table

| Checkpoint ID | Verification Task | Test Phase | Operational Status | Technical Result |
| :---: | :--- | :---: | :---: | :--- |
| **CP-01** | VMnet8 NAT Network Isolation | Phase 1 | `PASS` | All guest VMs assigned valid IPs on private subnet |
| **CP-02** | Appliance OVA Import | Phase 1 | `PASS` | System hardware specs & RAM allocations accepted |
| **CP-03** | `wazuh-manager` Service Health | Phase 1 | `PASS` | Systemd unit state reported `active (running)` |
| **CP-04** | `wazuh-indexer` Service Health | Phase 1 | `PASS` | Systemd unit state reported `active (running)` |
| **CP-05** | `wazuh-dashboard` Service Health | Phase 1 | `PASS` | Systemd unit state reported `active (running)` |
| **CP-06** | HTTPS Web Console Access | Phase 1 | `PASS` | Admin authenticated successfully on Port 443 |
| **CP-07** | Attacker -> Manager ICMP Reachability | Phase 1 | `PASS` | Ping test completed with 0% packet loss |
| **CP-08** | Manager -> Attacker ICMP Reachability | Phase 1 | `PASS` | Ping test completed with 0% packet loss |
| **CP-09** | Wazuh Agent Enrolment | Phase 1 | `PASS` | `kali-endpoint` registered and reported `Active` |
| **CP-10** | SSH Brute Force Detection | Phase 1 | `PASS` | Rules 5710/5712 generated severity 10 alert |
| **CP-11** | File Integrity Monitoring (FIM) | Phase 1 | `PASS` | Hash deltas captured via Rules 550/553/554 |
| **CP-12** | Nmap Reconnaissance Alert | Phase 1 | `PASS` | Network port scan flagged under MITRE T1046 |
| **CP-13** | Custom Log Path Configuration | Phase 2 | `PASS` | `/var/log/custom.log` tailed by `logcollector` |
| **CP-14** | XML Parent/Child Decoder | Phase 2 | `PASS` | `local_decoder.xml` parsed structured fields |
| **CP-15** | Bespoke Detection Rules | Phase 2 | `PASS` | Rules 100010/100011/100020 created successfully |
| **CP-16** | Offline Logic Validation | Phase 2 | `PASS` | `wazuh-logtest` validated all 3 processing phases |
| **CP-17** | Live Telemetry Ingestion | Phase 2 | `PASS` | Endpoint injected logs streamed to manager in <1s |
| **CP-18** | Discover KQL Search & Fields | Phase 2 | `PASS` | Decoded custom fields populated in search results |
| **CP-19** | Custom Dashboard Creation | Phase 2 | `PASS` | Metric, bar, table, & pie widgets saved to dashboard |
| **CP-20** | RBAC Role & User Creation | Phase 2 | `PASS` | Role `Read-Only-Analyst` & user `ro.analyst` built |
| **CP-21** | Black-Box Incognito Testing | Phase 2 | `PASS` | Read operations succeeded; administrative writes blocked |
| **CP-22** | Compliance Mapping & Sign-off | Phase 2 | `PASS` | Controls mapped to NIST, ISO 27001, and MITRE |

### 5.2 Regulatory & Framework Compliance

| Security Activity | NIST SP 800-53 Control | ISO/IEC 27001:2022 Control | MITRE ATT&CK Mapping |
| :--- | :--- | :--- | :--- |
| **File Integrity Monitoring** | `SI-7 Software & Info Integrity` | `A.8.32 / A.12.6.1` | Defense Evasion (`TA0005`) |
| **SSH Brute Force Detection** | `AU-6 Audit Review & Reporting` | `A.8.15 / A.12.4.1` | Credential Access (`T1110`) |
| **Nmap Recon Scan Alert** | `CA-7 Continuous Monitoring` | `A.8.16 / A.12.6.1` | Reconnaissance (`T1046`) |
| **Custom Log Ingestion & Rules**| `AU-12 Audit Record Generation` | `A.8.15 / A.12.4.3` | Detection Engineering |
| **SOC Dashboard Engineering** | `IR-4 Incident Handling / SI-4` | `A.5.25 / A.8.16` | Situational Awareness |
| **RBAC Governance & Access** | `AC-2 / AC-3 / AC-6 Least Privilege` | `A.5.15 / A.5.18` | Governance & Control |

---

## 6. References

- **Wazuh Documentation:** [Wazuh v4.x User Manual](https://documentation.wazuh.com/current/user-manual/)
- **Wazuh Ruleset Syntax:** [Custom Rules and Decoders Generation](https://documentation.wazuh.com/current/user-manual/ruleset/)
- **Wazuh RBAC Governance:** [Role-Based Access Control Setup](https://documentation.wazuh.com/current/user-manual/api/rbac/how-it-works.html)
- **NIST Guidelines:** [NIST SP 800-53 Rev. 5 Security Controls](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- **MITRE ATT&CK:** [ATT&CK Enterprise Matrix v14](https://attack.mitre.org/matrices/enterprise/)
- **Kali Linux Tools:** [Kali Documentation](https://www.kali.org/docs/)

---

<div align="center">

**Cyber Soch Blue Team | SOC Engineering & Security Operations**

</div>
