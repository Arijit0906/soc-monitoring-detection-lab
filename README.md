# Splunk - SOC Monitoring & Threat Detection Lab

## 📌 Project Overview & Objective
This self-directed portfolio project demonstrates the deployment of a centralized logging infrastructure tailored for security operations. By configuring a centralized SIEM platform, this environment bridges the gap between offensive attack emulation and defensive log analysis.

The primary objective is to simulate real-world cyber attacks from a dedicated offensive workstation and utilize localized agent forwarding to aggregate endpoint telemetry. This central repository allows for the creation of targeted detection rules, custom Splunk Search Processing Language (SPL) queries, and rigorous incident investigations.

---

## 🏗️ Core Architecture & Environment Components
The environment is structured into distinct functional zones to separate the security analytics layer, the offensive attack platform, and the targeted victim endpoints.

| Component Role | Operating System / Platform | Software & Agents Deployed | Telemetry & Logs Forwarded | Purpose & Function |
| :--- | :--- | :--- | :--- | :--- |
| **SIEM & Analytics Platform** | Kali Linux VM | • Splunk Enterprise<br>• Centralized Receiver Engine | • Inbound Endpoint Telemetry | Hosts the central Splunk Web console for log aggregation while serving as the analyst workstation. |
| **Adversary Platform** | Kali Linux VM | • Offensive Toolsets (Hydra, xfreerdp) | • Local Auditing Logs | Serves as the adversary machine to launch targeted exploits against the internal network subnet. |
| **Endpoint Target** | Windows 10 VM | • Splunk Universal Forwarder<br>• Native Windows Event Logging | • Windows Event Logs (Security, System, Application) | Acts as the primary victim for attack emulation, generating verbose endpoint telemetry to record malicious behavior. |

---

## 🌐 Network & Data Architecture Diagram

Data flows natively from both the Windows 10 target and the local Kali Linux subsystem via individual Splunk Universal Forwarders. These agents parse and securely route distinct event telemetry over TCP port 9997 into the centralized Splunk Enterprise indexer.

https://github.com/user-attachments/assets/746684ad-c4bd-407e-9a56-826ef7e2987f

---
## 🚀 Deployed Security Projects & Investigations

This repository serves as a live portfolio housing multiple defensive security use-cases. Each directory contains dedicated attack emulation steps, custom SIEM detection playbooks (SPL), and formal analyst incident reports.

### 📁 [Project 1: RDP Intrusion Lifecycle & Persistence (Completed)](./rdp-brute-force-detection/)
* **Focus Area:** Credential Access, Initial Access, & Active Persistence.
* **Core Analytics:** Catching high-velocity `EventCode=4625` (Logon Type 3) network brute-forcing, isolating the precise pivot to a successful `EventCode=4624` (Logon Type 10) Remote Interactive session, and tracking subsequent rogue account creation (`EventCode=4720`/`4732`) inside the local Administrators security tables.
* **Status:** 🟢 **Active Use-Case**

### 📁 Project 2: Advanced Endpoint Detection via Microsoft Sysmon (Upcoming)
* **Focus Area:** Living-off-the-Land (LotL) Command Execution & Discovery.
* **Core Analytics:** Transitioning host auditing into deep process monitoring using Sysmon `EventCode=1`. Hunting for anomalous process child/parent hierarchies (e.g., cmd/PowerShell spawned via web servers or malicious payloads) and isolating obscured command-line string arguments.
* **Status:** 🟡 **In Development Phase**

### 📁 Project 3: Automated Alert Tuning & Threshold Engineering (Upcoming)
* **Focus Area:** Detection Optimization & Reducing Alert Fatigue.
* **Core Analytics:** Writing progressive SPL tracking calculations using statistical moving averages to mathematically separate standard administrative maintenance patterns from true anomalous event floods.
* **Status:** ⚪ **Planned**
