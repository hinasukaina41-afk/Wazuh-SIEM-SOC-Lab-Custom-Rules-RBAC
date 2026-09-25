# 🛡️ Comprehensive SOC Security Task Report
### Virtual Lab Deployment, Real-Time Monitoring, Custom Log Decoders & Dashboard Engineering
*“Proactive Defense through Integrated Security Monitoring”*

[![Wazuh](https://img.shields.io/badge/SIEM-Wazuh%20v4.x-blue.svg)](https://wazuh.com/)
[![Hypervisor](https://img.shields.io/badge/Hypervisor-VMware%20Workstation%20Pro-orange.svg)](https://www.vmware.com/)
[![Offensive OS](https://img.shields.io/badge/Offensive%20OS-Kali%20Linux-red.svg)](https://www.kali.org/)
[![Compliance](https://img.shields.io/badge/Compliance-NIST%20800--53%20%7C%20ISO%2027001-green.svg)](#4-summary-table--compliance-matrix)
[![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-ff69b4.svg)](https://attack.mitre.org/)

---

## 📋 Metadata & Author Information
* **Author:** Hina Sukaina
* **Team:** Cyber Soch Blue Team
* **Domain:** SOC Security Engineering
* **Date:** 2026-09-15
* **Version:** v1.0
* **Core Domains:** SOC BLUE TEAM • THREAT DETECTION • SIEM ENGINEERING • RBAC GOVERNANCE

---

## 📑 Table of Contents
1. [Executive Summary & Objective Overview](#1-executive-summary--objective-overview)
   - [1.1 Executive Summary](#11-executive-summary)
   - [1.2 Objective Overview (Phase 1 & Phase 2)](#12-objective-overview)
2. [Virtual Lab Setup & Security Monitoring Fundamentals](#2-virtual-lab-setup--security-monitoring-fundamentals)
   - [2.1 Network Architecture & Flow Diagram](#21-vmware-workstation-setup--network-architecture)
   - [2.2 Appliance Import & Boot Verification](#22-kali-linux--wazuh-ova-import--boot-verification)
   - [2.3 Wazuh Stack Services Status Verification](#23-wazuh-stack-services-status-verification)
   - [2.4 Accessing Dashboard over HTTPS](#24-accessing-the-wazuh-web-dashboard-over-https)
   - [2.5 Bidirectional Network Connectivity Test](#25-bidirectional-network-connectivity-test)
   - [2.6 Agent Installation & Enrollment](#26-installing-the-wazuh-agent-on-the-endpoint)
   - [2.7 Real-Time Security Event Generation & Alerting](#27-security-event-generation--real-time-alert-detection)
     - [2.7.1 SSH Brute Force Simulation](#271-ssh-brute-force-simulation)
     - [2.7.2 File Integrity Monitoring (FIM) Test](#272-file-integrity-monitoring-fim-test)
     - [2.7.3 Nmap Network Scan Detection](#273-nmap-network-scan-detection)
3. [Custom Log Decoder, Detection Rules, Dashboards & RBAC](#3-custom-log-decoder-detection-rules-dashboards--rbac)
   - [3.1 Custom Log Source Configuration (`ossec.conf`)](#31-configuring-the-custom-log-source-on-the-agent-ossecconf)
   - [3.2 Custom XML Decoder Engineering (`local_decoder.xml`)](#32-engineering-the-custom-decoder-local_decoderxml)
   - [3.3 Custom Detection Rules Engineering (`local_rules.xml`)](#33-writing-custom-detection-rules-local_rulesxml)
   - [3.4 Manager Restart & `wazuh-logtest` Validation](#34-restarting-the-wazuh-manager--logic-validation-wazuh-logtest)
   - [3.5 End-to-End Live Log Generation](#35-generating-live-test-log-entries-on-the-endpoint)
   - [3.6 Discover Search & KQL Verification](#36-verifying-live-alerts-discover-searches-and-visualisations)
   - [3.7 Analyst Dashboard Engineering](#37-dashboard-setup--custom-widget-integration)
   - [3.8 Least-Privilege RBAC Role Setup](#38-read-only-analyst-role-rbac-setup)
   - [3.9 Incognito Session Boundary Verification](#39-incognito-browser-access-verification)
4. [Summary Table & Compliance Matrix](#4-summary-table--compliance-matrix)
   - [4.1 Checkpoint Summary Table (All 22 Verification Points)](#41-summary-table)
   - [4.2 Enterprise Compliance Matrix](#42-compliance-matrix)
5. [References](#5-references)

---

## 1. Executive Summary & Objective Overview

### 1.1 Executive Summary
This report documents the design, deployment, and operational validation of an isolated virtual security laboratory and the subsequent engineering of custom log ingestion, parsing, detection, visualisation, and access-control capabilities within the Wazuh unified SIEM/XDR platform. The laboratory was constructed on VMware Workstation Pro using a NAT-segmented virtual network hosting a Kali Linux offensive workstation and a Wazuh all-in-one appliance acting as the blue-team monitoring plane. The environment allows adversary tradecraft to be executed and observed end-to-end without exposing production assets or violating legal boundaries.

A Security Information and Event Management platform is the analytical backbone of a modern Security Operations Centre. It aggregates heterogeneous telemetry, normalises it into a consistent schema, correlates it against a curated detection ruleset, and surfaces prioritised alerts to analysts. Without centralised, normalised, and enriched logging, detection engineering, incident triage, threat hunting, and compliance reporting are not achievable at scale.

The task established the foundational layer: hypervisor networking, appliance import and boot verification, Wazuh manager/indexer/dashboard service health, HTTPS dashboard access, bidirectional connectivity assurance, agent enrolment, and validation of out-of-the-box detection through three representative attack simulations — SSH brute force, file integrity tampering, and Nmap reconnaissance. The task then extended the platform beyond default coverage: a bespoke application log source was onboarded, a custom XML decoder was engineered to parse its non-standard format, custom detection rules were authored and validated with wazuh-logtest, live alerts were confirmed in Discover, a purpose-built dashboard with analyst-focused widgets was created, and a least-privilege Read-Only-Analyst RBAC role was implemented and independently verified in an incognito session.

All twenty-two verification checkpoints completed successfully. The resulting environment demonstrates an operational detection lifecycle — collect, decode, detect, visualise, and govern access — mapped to NIST SP 800-53, ISO/IEC 27001:2022, and MITRE ATT&CK.

### 1.2 Objective Overview

#### 1.2.1 Phase 1 Objectives — Virtual Lab & Monitoring Fundamentals
1. Configure a VMware Workstation NAT subnet providing host-to-guest and guest-to-guest reachability with controlled outbound internet access.
2. Import and boot the Kali Linux and Wazuh OVA virtual appliances and verify successful initialisation and IP assignment.
3. Validate the health of the Wazuh stack services: `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard`.
4. Access the Wazuh web dashboard over HTTPS and complete authenticated login.
5. Prove bidirectional ICMP connectivity between the Kali attacker (`192.168.44.130`) and the Wazuh monitor (`192.168.10.187`).
6. Install and enrol a Wazuh agent on a monitored endpoint and confirm an Active state on the manager.
7. Generate authentic security events (SSH brute force, file integrity change, network port scan) and confirm real-time alerting.

#### 1.2.2 Phase 2 Objectives — Custom Decoder, Rules, Dashboards & RBAC
1. Onboard a non-standard application log file as a custom log source via the agent's `ossec.conf`.
2. Engineer a parent/child custom decoder in `local_decoder.xml` to extract structured fields from the raw log format.
3. Author custom detection rules in `local_rules.xml` (rule IDs `>= 100000`) referencing the custom decoder.
4. Restart the manager and validate decoder and rule logic deterministically using `wazuh-logtest`.
5. Generate live log entries on the endpoint and confirm end-to-end ingestion.
6. Verify alerts in the dashboard, filter them in Discover by custom rule ID, and inspect decoded fields.
7. Build and save a custom dashboard containing alert-count, timeline, and top-source visualisations.
8. Create a least-privilege Read-Only-Analyst role and an associated user under Wazuh RBAC.
9. Independently verify read-only enforcement from an incognito browser session.

---

## 2. Virtual Lab Setup & Security Monitoring Fundamentals

### 2.1 VMware Workstation Setup & Network Architecture

The laboratory is hosted on VMware Workstation Pro (Type 2 hypervisor). All guests are attached to a NAT-backed virtual network (`VMnet8`), which is served by the VMware NAT service and the VMware DHCP service.

#### Network Topology Diagram
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
