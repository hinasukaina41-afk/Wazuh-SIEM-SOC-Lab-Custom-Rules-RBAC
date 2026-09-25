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
🎯 ObjectivesThe primary objectives of this project were to:Build an isolated SOC laboratory environment.Deploy and configure a Wazuh Manager.Connect a Kali Linux endpoint as a Wazuh Agent.Validate endpoint-to-manager communication.Generate controlled security events.Test authentication-failure detection.Test File Integrity Monitoring (FIM).Generate network reconnaissance activity using Nmap.Develop a custom Wazuh decoder.Develop a custom Wazuh detection rule.Map the detection to MITRE ATT&CK T1110 — Brute Force.Validate the detection using wazuh-logtest.Verify the alert in the Wazuh Dashboard.Build a saved search and visualization.Configure a read-only analyst role using Wazuh RBAC.🏗️ Lab ArchitecturePlaintext┌─────────────────────────────────────────┐
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
Network ConfigurationEntityRoleIP AddressVMware VMnet8NAT Gateway192.168.44.2Kali LinuxAttacker / Monitored Endpoint192.168.44.130Wazuh ApplianceAll-in-One SIEM Monitoring Stack192.168.10.187🧰 Technology StackTechnologyPurpose🛡️ WazuhSIEM, log analysis, detection and monitoring🐉 Kali LinuxSecurity testing endpoint💻 VMware WorkstationVirtualized laboratory infrastructure🔍 Nmap / HydraNetwork reconnaissance & brute force testing📊 Wazuh DashboardAlert investigation and visualization🧪 wazuh-logtestDecoder and rule validation🎯 MITRE ATT&CKAdversary technique mapping🔐 RBACLeast-privilege SOC access control⚡ Phase 1 — Virtual SOC Lab Setup1. Virtualization & Infrastructure DeploymentThe lab environment was created on VMware Workstation Pro using an isolated NAT subnet (VMnet8).2. Wazuh DeploymentThe Wazuh All-in-One OVA was imported and configured with required hardware resources (4 vCPU, 8 GB RAM).The deployment consists of:PlaintextWazuh Agent (Kali Linux)
   │
   ▼
Wazuh Manager
   ├── Wazuh Indexer
   └── Wazuh Dashboard
The three core Wazuh services were verified as active:Bashsudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
3. Connectivity VerificationBidirectional communication between the endpoint and the Wazuh manager was verified using ICMP testing:Bash# Ping from Kali endpoint to Wazuh server
ping -c 4 192.168.10.187

# Ping from Wazuh server to Kali endpoint
ping -c 4 192.168.44.130
📥 Phase 2 — Wazuh Agent Deployment1. Agent InstallationThe Wazuh Agent was installed on the Kali Linux system and configured to communicate with the Wazuh Manager:Bashsudo WAZUH_MANAGER="192.168.10.187" WAZUH_AGENT_NAME="kali-endpoint" apt-get install -y wazuh-agent
sudo systemctl enable --now wazuh-agent
2. Enrollment VerificationAgent enrollment was confirmed on the Wazuh Manager CLI:Bashsudo /var/ossec/bin/agent_control -l
🚨 Phase 3 — Out-of-the-Box Security MonitoringTo validate basic security monitoring, three security events were simulated on the endpoint:Authentication Failure (SSH Brute Force): Simulated using Hydra.Bashhydra -l analyst -P /usr/share/wordlists/rockyou.txt ssh://192.168.44.130 -t 4
File Integrity Monitoring (FIM): Triggered by altering system files under /etc.Bashsudo touch /etc/rogue_marker.conf
Network Reconnaissance: Generated using Nmap port scanning.Bashsudo nmap -sS -T4 -p 1-1024 192.168.44.130
🧩 Phase 4 — Custom Log Decoder & Rule Engineering1. Custom Log File SetupA custom application log source was registered in the agent's /var/ossec/etc/ossec.conf:XML<ossec_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/custom.log</location>
  </localfile>
</ossec_config>
2. Custom XML Decoder DevelopmentCreated in /var/ossec/etc/decoders/local_decoder.xml:XML<decoder name="customapp">
  <prematch>CUSTOMAPP</prematch>
</decoder>

<decoder name="customapp-fields">
  <parent>customapp</parent>
  <regex offset="after_parent">severity=(\S+) user=(\S+) src_ip=(\S+) action=(\S+)</regex>
  <order>severity, user, srcip, action</order>
</decoder>
3. Custom Detection Rule DevelopmentCreated in /var/ossec/etc/rules/local_rules.xml (Mapped to MITRE ATT&CK T1110):XML<group name="customapp,">
  <rule id="100000" level="3">
    <decoded_as>customapp</decoded_as>
    <description>CUSTOMAPP: Event received</description>
  </rule>

  <rule id="100010" level="7">
    <if_sid>100000</if_sid>
    <field name="action">LOGIN_FAILED</field>
    <description>CUSTOMAPP: Login failure for user $(user) from $(srcip)</description>
    <mitre><id>T1078</id></mitre>
  </rule>

  <rule id="100011" level="10" frequency="5" timeframe="60">
    <if_matched_sid>100010</if_matched_sid>
    <same_field>srcip</same_field>
    <description>CUSTOMAPP: Brute force detected - 5 failures from $(srcip) in 60s</description>
    <mitre><id>T1110</id></mitre>
  </rule>
</group>
4. Rule Validation using wazuh-logtestThe custom decoder and detection rules were validated using the log testing tool:Bashsudo /var/ossec/bin/wazuh-logtest
📊 Phase 5 — Dashboard Engineering & RBAC Governance1. Saved Search & VisualizationFiltered custom events in Discover using DQL:Code snippetrule.id: 100011
Built a custom dashboard (CUSTOMAPP - Cyber Soch Analyst View) displaying alert metrics, timeline charts, and top source IP tables.2. Read-Only RBAC ImplementationCreated a least-privilege role (Read-Only-Analyst) mapped to user ro.analyst. Verified read-only enforcement in an Incognito private browsing window:Allowed: Viewing alerts, dashboards, and security events.Denied (HTTP 403): Editing rules, decoders, or agent configurations.📑 Summary & Compliance MatrixActivityNIST SP 800-53ISO/IEC 27001MITRE ATT&CKFile Integrity MonitoringSI-7A.8.32TA0005 — Defense EvasionSSH Brute ForceAU-6A.8.15Credential Access (T1110)Nmap Scan DetectionCA-7A.8.16Reconnaissance (T1046)Custom Rules EngineAU-12A.12.4Detection EngineeringRBAC GovernanceAC-2 / AC-3A.5.15Security Governance👤 Author InformationHina SukainaCyber Security Analyst & Threat HunterTeam: Cyber Soch Blue TeamDomain: SOC Security Engineering
