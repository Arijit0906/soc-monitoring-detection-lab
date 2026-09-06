# Splunk - SOC Monitoring & Threat Detection Lab

## 📌 Project Overview & Objective
This self-directed portfolio project demonstrates the deployment of a centralized logging infrastructure tailored for security operations. By configuring a centralized SIEM platform, this environment bridges the gap between offensive attack emulation and defensive log analysis. 

The primary objective is to simulate real-world cyber attacks from a dedicated offensive workstation and utilize localized agent forwarding to aggregate endpoint telemetry. This central repository allows for the creation of targeted detection rules, custom Splunk Search Processing Language (SPL) queries, and rigorous incident investigations.

---

## 🏗️ Core Architecture & Environment Components

The environment is structured into distinct functional zones to separate the security analytics layer, the offensive attack platform, and the targeted victim endpoints. 

| Component Role | Operating System / Platform | Software & Agents Deployed | Telemetry & Logs Forwarded | Purpose & Function |
| :--- | :--- | :--- | :--- | :--- |
| **SIEM & Attack Platform**<br>*(Attack Machine)* | Kali Linux VM | • Splunk Enterprise (v10.4.1)<br>• Splunk Universal Forwarder<br>• Offensive Toolsets | • Linux System Logs (`/var/log/`)                   | Hosts the central Splunk Web console for log aggregation while serving as the adversary machine to launch exploits. Monitors its own system logs for local auditing. |
| **Endpoint Victim**<br>*(Victim Machine)* | Windows 10 VM | • Splunk Universal Forwarder<br>• Microsoft Sysmon<br>• Native Windows Event Logging | • Microsoft Sysmon Logs<br>• Windows Event Logs (Security, System, Application) | Acts as the primary target for attack emulation, generating verbose endpoint telemetry to track and record malicious behavior. |


---

## 🌐 Network & Data Architecture Diagram

Data flows natively from both the Windows 10 target and the local Kali Linux subsystem via individual Splunk Universal Forwarders. These agents parse and securely route distinct event telemetry over TCP port 9997 into the centralized Splunk Enterprise indexer.
https://github.com/user-attachments/assets/746684ad-c4bd-407e-9a56-826ef7e2987f

