# Enterprise Endpoint Detection & Automated Active Response Lab

## Project Overview
This project simulates an enterprise Security Operations Center (SOC) telemetry and detection pipeline in a virtualized lab environment. Using Wazuh SIEM/XDR, an endpoint was configured to forward real-time security events to an analytics manager. An authentication brute-force attack was simulated against the monitored host, triggering out-of-the-box and custom detection rules mapped to the MITRE ATT&CK framework, followed by automated host containment via Wazuh Active Response.

---

## Architecture & Lab Topology

### Components
* **SIEM / Manager:** Wazuh Server & Dashboard (VirtualBox VM)
* **Monitored Endpoint:** Linux Server (Ubuntu) running `wazuh-agent`
* **Adversary Machine:** Attack platform executing simulated authentication attacks
* **Network:** Host-Only / NAT Network isolation

---

## Threat Scenario & MITRE ATT&CK Mapping

* **Tactic:** Credential Access ([TA0006](https://attack.mitre.org/tactics/TA0006/))
* **Technique:** Brute Force - Password Guessing ([T1110.001](https://attack.mitre.org/techniques/T1110/001/))
* **Execution:** High-frequency SSH authentication attempts targeting system accounts to discover valid credentials.

---

## Implementation & Execution

### 1. Agent Enrollment & Telemetry Verification
* Installed and registered the Wazuh agent using enrollment tokens via CLI.
* Configured log collection inside `/var/ossec/etc/ossec.conf` targeting `/var/log/auth.log`.
* Verified bidirectional communication on TCP ports 1514 (events) and 1515 (registration).

---

### 2. Attack Simulation
* Executed an automated dictionary attack against the target endpoint over SSH:
  ```bash
  hydra -l user -P rockyou.txt ssh://<TARGET_IP>

​Generated repeated failed authentication spikes within a 60-second window to simulate anomalous credential guessing.

3. Log Ingestion & Detection
​

The monitored endpoint logged failed attempts in /var/log/auth.log.
​Wazuh Manager ingested the events, triggering built-in rules:
​Rule ID 5404: Three failed attempts to run sudo
​Rule ID 5557: unix_chkpwd: Password check failed
​Rule ID 5712: sshd: Brute force trying to get access to the system

<img width="1366" height="768" alt="Screenshot (9)" src="https://github.com/user-attachments/assets/6e59286a-52de-4066-9fa1-79ac5f7c54bb" />

​4. Automated Active Response (Containment)
​To reduce analyst mean-time-to-respond (MTTR), an Active Response rule was bound to the detection threshold:

Configured ossec.conf on the Wazuh Manager:

<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712</rules_id>
  <timeout>600</timeout>
</active-response>



Outcome: When the brute-force threshold fired, the Wazuh agent executed the firewall-drop script, dropping ingress traffic from the attacker IP.



Verification: Confirmed the block via /var/ossec/logs/active-responses.log.



<img width="1366" height="768" alt="Screenshot (12)" src="https://github.com/user-attachments/assets/dd7eef70-f315-4f9d-a073-9cd266a27050" />


Key Skills Demonstrated
​

SIEM Administration: Agent deployment, port configuration, and telemetry pipeline verification.
​
Log Triage & Analysis: Parsing Linux authentication logs (auth.log) to identify brute-force patterns.
​
Detection Engineering: Alert categorization and MITRE ATT&CK technique mapping.
​
Incident Containment: Automating endpoint response controls (firewall-drop) via host agents.



  
