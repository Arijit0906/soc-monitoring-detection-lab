# My Splunk Investigation and Triage Playbook

This document details how I routed logs from my Windows 10 VM into Splunk, followed by the step-by-step investigation I conducted to uncover the RDP attack lifecycle based on my live SIEM data.

---

## Data Pipeline Setup: Getting Logs into Splunk

To start gathering evidence, I set up a quick data pipeline between my virtual machines:
* **Splunk Web Setup:** On my Kali machine (`192.168.100.1`), I enabled a network receiving port on TCP `9997` and created an index named `endpoint`.
* **Forwarder Setup:** On my Windows 10 VM (`192.168.100.2`), I installed the Splunk Universal Forwarder and directed it to my Kali IP.
* **Input Routing:** I configured the forwarder's local `inputs.conf` file to actively grab the Windows Security event stream (`WinEventLog://Security`). 

Once the service restarted, logs immediately began populating my dashboard application.

---

## Forensic Investigation & Incident Triage Steps

### Step 1: Initial Discovery of Failed Logins
While checking my environment, I noticed a sudden sequence of failed authentication attempts hitting the system. I initiated my query targeting logon failures:

```splunk
index=* source="WinEventLog:Security" EventCode=4625
```
<img width="1635" height="740" alt="image" src="https://github.com/user-attachments/assets/9f3eb278-106f-48d8-9618-50bc7fac1bd3" />

I looked closely at a random failed log entry from the results and found an unknown source IP address **`192.168.100.1`** with the workstation name **`kali`** aggressively trying to access the user account **`Finn`**.
<img width="1021" height="945" alt="image" src="https://github.com/user-attachments/assets/a70601da-d348-4755-be95-74769b007fb1" />

---

### Step 2: Mapping the Scope of the Brute-Force Attack
To understand the full scale of this malicious activity, I ran a summary query to identify how many unique rogue IPs were involved and which specific local user accounts they were targeting:

```splunk
index=* sourcetype="WinEventLog:Security" EventCode=4625
| stats dc(Source_Network_Address) as Distinct_IP_Count, values(Source_Network_Address) as Attacking_IP_List, count as Total_Failed_Attempts by Account_Name
| sort - Distinct_IP_Count
```
<img width="2076" height="423" alt="image" src="https://github.com/user-attachments/assets/15235dd6-01e7-4d48-9456-07f1096a614a" />


#### Why I structured this command:
* **`by Account_Name`**: Groups the results into clean rows for each targeted user profile.
* **`dc(Source_Network_Address)`**: Calculates the exact distinct count of unique attacking IPs.
* **`values(Source_Network_Address)`**: Lists the actual source IP locations.

#### My Findings:
As shown in my Splunk dashboard screenshot, the rogue IP **`192.168.100.1`** made **14 distinct failed attempts** to guess the password for the account **`Finn`**.

---

### Step 3: High-Velocity Time Analysis
Next, I needed to check the exact timing of these events to determine if this was a human typo or an automated password-guessing script:

```splunk
index=* source="WinEventLog:Security" EventCode=4625 Source_Network_Address=192.168.100.1 
| stats count by _time
```
<img width="1667" height="589" alt="image" src="https://github.com/user-attachments/assets/561ff9ee-41f2-4f15-b54e-6e104018e627" />

#### My Findings:
Looking at my timeline statistics, the logs reveal an automated pattern: **10 failed login attempts occurred within a single minute** (all hitting sequentially between `16:07:12` and `16:07:56`). This confirmed a dictionary brute-force attack was underway.

---

### Step 4: Tracking the Successful Account Takeover
Since the attacker was using a password list, I checked if any of their attempts ultimately succeeded. Because Event ID 4624 denotes a successful logon but doesn't always guarantee a human is at the keyboard, I checked for remote interactive sessions originating from the attacker's footprint:

```splunk
index=* source="WinEventLog:Security" EventCode=4624 Source_Network_Address=192.168.100.1
| stats count by _time
```
<img width="2095" height="520" alt="image" src="https://github.com/user-attachments/assets/dcbad524-db8c-47c8-ac16-2dc4edb49d1d" />

The query confirmed that the logon state shifted to success, proving the attacker successfully guessed the correct credentials. 

---
#### 🚨 Critical Triage Pivot: Checking for a Remote Logon Session
To determine if this was a background background network check (`Logon_Type=3`) or an active graphical takeover, I checked the logon mechanics. Finding an explicit **`Logon_Type=10` (Remote Interactive)** confirmed the attacker successfully opened a full Remote Desktop window to control the machine manually.

```splunk
Index=* source="WinEventLog:Security" EventCode=4624 Source_Network_Address=192.168.100.1  Logon_Type=10 | stats count by _time
```
<img width="2074" height="346" alt="image" src="https://github.com/user-attachments/assets/34807143-9165-491b-bcfd-be95f0cd0824" />

---

### Step 5: Uncovering Post-Compromise Backdoors
With administrative access secured, I hunted for suspicious actions the attacker might have taken to ensure permanent access to my Windows VM. I audited the index for account provisioning and group modification events:

```splunk
index=* (EventCode=4720 OR EventCode=4732)
| eval Action=if(EventCode==4720, "User Account Created", "Added to Local Admin Group")
| table _time, ComputerName, Action, Security_ID
```
<img width="2054" height="534" alt="image" src="https://github.com/user-attachments/assets/9b600f50-33a9-4b0e-a2e5-3b2e32708184" />

#### My Findings:
My forensic search caught three highly critical modifications matching the breach timeline:
1. **User Account Created (`EventCode=4720`):** A new unauthorized local profile was added to the operating system.
2. **Added to Local Admin Group (`EventCode=4732`):** That exact backdoor profile was elevated directly into the local `Administrators` security group, granting the attacker full root access.

---

## Incident Escalation Notice

🚨 **CRITICAL FINDING:** This investigation confirms a verified **True Positive (TP)** remote system compromise. The rapid succession of 14 network login failures followed by a successful interactive logon session and subsequent unauthorized administrative user creation proves malicious activity. 

Pursuant to standard incident response protocols, this event is being **escalated to the Tier-2 Incident Response Team** for immediate host network isolation, account credential revocation, and active system containment. 

---

### Project Conclusion
This project successfully demonstrates a complete end-to-end security workflow—from building a functional data pipeline and mimicking real-world threat actors to writing structured analytical queries that track an attack down to the exact second. This lab successfully proves my practical ability to monitor infrastructure, hunt for anomalies, and isolate malicious intent within an enterprise environment.

