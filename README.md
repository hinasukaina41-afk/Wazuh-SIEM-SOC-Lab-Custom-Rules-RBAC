# 🛡️ Comprehensive SOC Security Task Report: Wazuh SIEM Lab

**🔎 Virtual Lab Deployment • Real-Time Monitoring • Custom Decoders & Rules • Dashboard Engineering • RBAC**

---

## 🛰️ Project Overview

This project documents the design, deployment, and operational validation of an isolated virtual security laboratory and the subsequent engineering of custom log ingestion, parsing, detection, visualisation, and access-control capabilities within the **Wazuh unified SIEM/XDR platform**.

The laboratory was constructed on **VMware Workstation Pro** using a NAT-segmented virtual network hosting a **Kali Linux** offensive workstation and a **Wazuh all-in-one appliance** acting as the blue-team monitoring plane. The environment allows adversary tradecraft to be executed and observed end-to-end without exposing production assets or violating legal boundaries.

The project successfully demonstrates an operational detection lifecycle — **collect, decode, detect, visualise, and govern access** — mapped to NIST SP 800-53, ISO/IEC 27001:2022, and MITRE ATT&CK.

---

## 🎯 Objectives

The primary objectives of this project were to:

### Phase 1 — Virtual Lab & Monitoring Fundamentals
*   Configure a VMware Workstation NAT subnet providing host-to-guest and guest-to-guest reachability.
*   Import and boot the Kali Linux and Wazuh OVA virtual appliances.
*   Validate the health of the Wazuh stack services: `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard`.
*   Access the Wazuh web dashboard over HTTPS and complete authenticated login.
*   Prove bidirectional ICMP connectivity between the Kali attacker and the Wazuh monitor.
*   Install and enrol a Wazuh agent on a monitored endpoint and confirm an **Active** state.
*   Generate authentic security events (SSH brute force, file integrity change, network port scan) and confirm real-time alerting.

### Phase 2 — Custom Decoder, Rules, Dashboards & RBAC
*   Onboard a non-standard application log file as a custom log source via the agent's `ossec.conf`.
*   Engineer a parent/child custom decoder in `local_decoder.xml` to extract structured fields.
*   Author custom detection rules in `local_rules.xml` (rule IDs >= 100000) referencing the custom decoder.
*   Validate decoder and rule logic deterministically using `wazuh-logtest`.
*   Verify live alerts in the dashboard, filter them in **Discover**, and inspect decoded fields.
*   Build and save a custom dashboard containing alert-count, timeline, and top-source visualisations.
*   Create a least-privilege **Read-Only-Analyst** role and an associated user under Wazuh RBAC.
*   Independently verify read-only enforcement from an incognito browser session.

---

## 🏗️ Lab Architecture

The laboratory is hosted on **VMware Workstation Pro** (Type 2 hypervisor). All guests are attached to a NAT-backed virtual network (VMnet8), which is served by the VMware NAT service and the VMware DHCP service.

```text
                    ┌─────────────────────────────┐
                    │        HOST SYSTEM          │
                    │      (Windows/Linux)        │
                    └──────────────┬──────────────┘
                                   │
                         VMware NAT Network (VMnet8)
                           192.168.xx.xx/xx
                                   │
                     ┌─────────────┴─────────────┐
                     │                           │
                     ▼                           ▼
           ┌──────────────────┐         ┌────────────────────┐
           │   Kali Linux     │         │   Wazuh Server     │
           │                  │         │                    │
           │ 192.168.xx.xx    │         │ 192.168.xx.xx      │
           │                  │         │                    │
           │  Wazuh Agent     │────────▶│ Wazuh Manager      │
           │  Nmap            │  Logs   │ Wazuh Indexer      │
           │  Hydra           │         │ Wazuh Dashboard    │
           └──────────────────┘         └────────────────────┘
