# INCIDENT TRIAGE REPORT: INC-2026-001

* **Time of activity:**
  * **Incident ID:** INC-2026-001
  * **Date/Time Occurred:** 2026-09-22 16:07 UTC
  * **Date/Time Detected:** 2026-09-22 16:12 UTC *(Triggered via real-time Splunk alert 5 minutes post-activity)*
  * **Date/Time Reported:** 2026-09-22 16:42 UTC *(Escalated half an hour after triage)*
  * **Incident Handler / Owner:** Arijit / SOC Department
  * **Severity Level:** High

* **List of Affected Entities:** 
  * Target Host: Windows 10 Enterprise VM (`DESKTOP-QTJ31N5` / `192.168.100.2`)
  * Compromised Profile: Local Administrator account (`Finn`)
  * Network Interface: Remote Desktop Protocol service (TCP Port `3389`)

* **Reason for Classifying as True Positive:** The sequential pattern is entirely malicious. An external Linux machine (`kali` / `192.168.100.1`) generated 14 rapid login failures within one minute, immediately followed by a successful interactive connection and unauthorized system modifications. This rules out a normal user typo.

* **Reason for Escalating the Alert:** **CRITICAL SEVERITY.** The attacker successfully gained interactive graphical access and established permanent persistence by creating an unauthorized local account elevated to the local `Administrators` group. Immediate containment is required.

* **Recommended Remediation Actions:**
  1. Isolate the Windows 10 VM (`192.168.100.2`) from the network immediately.
  2. Terminate all active RDP sessions originating from the attacking IP (`192.168.100.1`).
  3. Disable the compromised `Finn` user account and force a global password reset.
  4. Manually delete the unauthorized `attacker_backdoor` account from the system.

* **List of Attack Indicators:**
  * **Event ID 4625 (Logon Type 3):** 14 rapid authentication failures from IP `192.168.100.1`.
  * **Event ID 4624 (Logon Type 10):** Successful remote interactive desktop takeover of the `Finn` profile.
  * **Event ID 4720:** Creation of an unauthorized local account named `attacker_backdoor`.
  * **Event ID 4732:** Elevation of the backdoor account into the local `Administrators` security group.

