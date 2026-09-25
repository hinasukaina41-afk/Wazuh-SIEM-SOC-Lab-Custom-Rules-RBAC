# 🛡️ Enterprise SOC Security Engineering & SIEM Analytics Lab
### 🔬 Virtual Lab Deployment, Real-Time Monitoring, Custom Log Decoders & Dashboard Engineering
> *“Proactive Defense through Integrated Security Monitoring”*

[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blue.svg?style=for-the-badge&logo=wazuh)](https://wazuh.com/)
[![Hypervisor](https://img.shields.io/badge/Hypervisor-VMware%20Workstation%20Pro-orange.svg?style=for-the-badge&logo=vmware)](https://www.vmware.com/)
[![Offensive OS](https://img.shields.io/badge/Attacker-Kali%20Linux-red.svg?style=for-the-badge&logo=kali-linux)](https://www.kali.org/)
[![Compliance](https://img.shields.io/badge/Compliance-NIST%20800--53%20%7C%20ISO%2027001-green.svg?style=for-the-badge)](#-summary-table--compliance-matrix)
[![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-ff69b4.svg?style=for-the-badge)](https://attack.mitre.org/)

---

## 👤 Metadata & Author Information
* 👩‍💻 **Author:** Hina Sukaina
* 👥 **Team:** Cyber Soch Blue Team
* 🛡️ **Domain:** SOC Security Engineering
* 📅 **Date:** 2026-09-15
* 🏷️ **Version:** v1.0
* 🎯 **Core Focus:** SOC BLUE TEAM • THREAT DETECTION • SIEM ENGINEERING • RBAC GOVERNANCE

---

## 📑 Table of Contents
- [📌 1. Executive Summary & Objective Overview](#-1-executive-summary--objective-overview)
  - [📑 1.1 Executive Summary](#-11-executive-summary)
  - [🎯 1.2 Objective Overview (Phase 1 & Phase 2)](#-12-objective-overview)
- [🖥️ 2. Virtual Lab Setup & Security Monitoring Fundamentals](#%EF%B8%8F-2-virtual-lab-setup--security-monitoring-fundamentals)
  - [📐 2.1 Network Architecture & Topology](#-21-vmware-workstation-setup--network-architecture)
  - [⚙️ 2.2 Kali Linux & Wazuh OVA Import](#%EF%B8%8F-22-kali-linux--wazuh-ova-import--boot-verification)
  - [🟢 2.3 Wazuh Stack Services Status Verification](#-23-wazuh-stack-services-status-verification)
  - [🔒 2.4 Accessing the Wazuh Web Dashboard over HTTPS](#-24-accessing-the-wazuh-web-dashboard-over-https)
  - [🌐 2.5 Bidirectional Network Connectivity Test](#-25-bidirectional-network-connectivity-test)
  - [📥 2.6 Installing the Wazuh Agent on the Endpoint](#-26-installing-the-wazuh-agent-on-the-endpoint)
  - [🚨 2.7 Security Event Generation & Real-Time Alerting](#-27-security-event-generation--real-time-alert-detection)
- [⚙️ 3. Custom Log Decoder, Detection Rules, Dashboards & RBAC](#%EF%B8%8F-3-custom-log-decoder-detection-rules-dashboards--rbac)
  - [📂 3.1 Custom Log Source Configuration (`ossec.conf`)](#-31-configuring-the-custom-log-source-on-the-agent-ossecconf)
  - [🧩 3.2 Custom XML Decoder Engineering (`local_decoder.xml`)](#-32-engineering-the-custom-decoder-local_decoderxml)
  - [⚡ 3.3 Custom Detection Rules Engineering (`local_rules.xml`)](#-33-writing-custom-detection-rules-local_rulesxml)
  - [🧪 3.4 Manager Restart & `wazuh-logtest` Validation](#-34-restarting-the-wazuh-manager--logic-validation-wazuh-logtest)
  - [📝 3.5 End-to-End Live Log Generation](#-35-generating-live-test-log-entries-on-the-endpoint)
  - [🔍 3.6 Discover Search & KQL Verification](#-36-verifying-live-alerts-discover-searches-and-visualisations)
  - [📊 3.7 Custom Analyst Dashboard Engineering](#-37-dashboard-setup--custom-widget-integration)
  - [🔐 3.8 Read-Only Analyst Role (RBAC) Setup](#-38-read-only-analyst-role-rbac-setup)
  - [🕵️‍♂️ 3.9 Incognito Session Boundary Verification](#%EF%B8%8F-39-incognito-browser-access-verification)
- [📊 4. Summary Table & Compliance Matrix](#-4-summary-table--compliance-matrix)
- [📚 5. References](#-5-references)

---

## 📌 1. Executive Summary & Objective Overview

### 📑 1.1 Executive Summary
This report documents the design, deployment, and operational validation of an isolated virtual security laboratory and the subsequent engineering of custom log ingestion, parsing, detection, visualisation, and access-control capabilities within the Wazuh unified SIEM/XDR platform. The laboratory was constructed on VMware Workstation Pro using a NAT-segmented virtual network hosting a Kali Linux offensive workstation and a Wazuh all-in-one appliance acting as the blue-team monitoring plane.

A Security Information and Event Management (SIEM) platform is the analytical backbone of a modern Security Operations Centre (SOC). It aggregates heterogeneous telemetry, normalises it into a consistent schema, correlates it against a curated detection ruleset, and surfaces prioritised alerts to analysts.

### 🎯 1.2 Objective Overview

#### 🎯 Phase 1 Objectives — Virtual Lab & Monitoring Fundamentals
* 🌐 **Network Setup:** Configure a VMware Workstation NAT subnet providing host-to-guest and guest-to-guest reachability.
* 🖥️ **Appliance Import:** Import and boot Kali Linux and Wazuh OVA virtual appliances with stable IP assignment.
* 🟢 **Service Health:** Validate `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard` status.
* 🔒 **Dashboard Access:** Authenticate onto the Wazuh web interface over HTTPS.
* ⚡ **Reachability Test:** Prove bidirectional ICMP reachability between Kali Attacker (`192.168.44.130`) and Wazuh Server (`192.168.10.187`).
* 📥 **Agent Enrollment:** Install and enrol a Wazuh agent on the monitored endpoint (`kali-endpoint`).
* 🚨 **Out-of-the-Box Detection:** Generate authentic security events (SSH Brute Force, File Integrity Monitoring, Nmap Scan).

#### 🎯 Phase 2 Objectives — Custom Decoder, Rules, Dashboards & RBAC
* 📂 **Custom Ingestion:** Onboard a custom log source `/var/log/custom.log` via agent `ossec.conf`.
* 🧩 **Decoder Engineering:** Build parent/child XML decoders in `local_decoder.xml`.
* ⚡ **Detection Rules:** Author custom rules in `local_rules.xml` (rule IDs `>= 100000`).
* 🧪 **Diagnostic Testing:** Validate logic deterministically using `wazuh-logtest`.
* 📝 **Live Telemetry:** Generate live logs on the endpoint and confirm real-time ingestion.
* 🔍 **Discover & KQL:** Verify and filter alerts in Discover using KQL queries.
* 📊 **Dashboard Creation:** Design custom widgets (metric, timeline, top sources, pie charts).
* 🔐 **RBAC Governance:** Create a least-privilege `Read-Only-Analyst` role and `ro.analyst` user.
* 🕵️‍♂️ **Incognito Audit:** Confirm read-only enforcement in an independent private session.

---

## 🖥️ 2. Virtual Lab Setup & Security Monitoring Fundamentals

### 📐 2.1 VMware Workstation Setup & Network Architecture

The environment is segmented inside VMware Workstation Pro on a dedicated NAT Subnet (`VMnet8`).

#### 🔀 Network Topology Flowchart
```text
                          ┌──────────────────────────────────────┐
                          │   Host System (Windows / Hypervisor) │
                          └──────────────────┬───────────────────┘
                                             │
                   ┌─────────────────────────┴─────────────────────────┐
                   │   VMware Workstation VMnet8 (NAT Subnet)          │
                   │   Subnet: 192.168.44.0/24 / Gateway: .2           │
                   └─────────────┬───────────────────────────┬─────────┘
                                 │                           │
           ┌─────────────────────▼───┐           ┌───────────▼─────────────────────┐
           │   Kali Linux Workstation │           │   Wazuh All-In-One Appliance    │
           │   (Offensive / Endpoint) │           │   (Blue Team Monitoring Plane)  │
           │   IP: 192.168.44.130     │ ◄───────► │   IP: 192.168.10.187            │
           │   (Wazuh Agent Enrolled) │ ICMP/1514 │   (Manager / Indexer / Dash)    │
           └─────────────────────────┘           └─────────────────────────────────┘
