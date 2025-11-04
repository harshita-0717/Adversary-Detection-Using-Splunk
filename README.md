


# 🛡️ Adversary Detection Lab using Splunk (SOC Simulation)

This project simulates a real-world Security Operations Center (SOC) environment, utilizing the Elastic/SIEM tool **Splunk** to detect, analyze, and report on cybersecurity attacks launched from an attacker machine.

---

### 🎯 Project Goal and Motivation

* **Goal:** To build a fully isolated virtual lab environment and develop detection rules (Splunk SPL) for common attack techniques, specifically focusing on the **MITRE ATT&CK Framework**.
* **Motivation:** Gain hands-on experience in **Blue Teaming (Detection)**, log analysis, threat hunting, and understanding the attacker perspective (Red Teaming) to improve security posture.

---

### ⚙️ Environment Setup and Configuration

This lab uses a minimal, two-machine setup connected via an isolated network.

| Component | Role | IP Address (Example) | OS/Software |
| :--- | :--- | :--- | :--- |
| **Kali Linux** | **Attacker (Red Team)** | `192.168.56.101` | Kali Linux VM |
| **Ubuntu VM** | **Target/Defender (Blue Team)** | `192.168.56.103` | Ubuntu Desktop/Server, Splunk Enterprise |

#### Network Verification
* **Configuration:** Both VMs were set to a **Host-Only Adapter** to ensure complete isolation from the external network.
* **Verification:** Connectivity was confirmed by successfully pinging the Ubuntu machine (`192.168.56.103`) from Kali.


#### Log Ingestion Configuration
* **Platform:** Splunk Enterprise was installed on the Ubuntu VM.
* **Data Source:** Configured Splunk to monitor critical Linux system logs for security analysis.
* **Index Used:** `os_logs`
* **Key Log Files Ingested:**
    * `/var/log/auth.log` (Crucial for login/authentication attempts)
    * `/var/log/syslog` (General system messages)

#### ✍️ Why This Configuration is Necessary
1.	**Source Type:** Setting a specific Source Type (like linux_secure) tells Splunk exactly how to parse the data, which means it knows where to find the timestamp, the host, and the message content, making searches faster and more accurate.
2.	**Index Name:** Using a dedicated Index (like os_logs) keeps your security data separate from Splunk's internal logs, allowing you to limit searches to only relevant data and improving both performance and organization, which is a key SOC standard.

