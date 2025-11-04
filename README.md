<img src="Images/front_1.png" alt="front_images" width="100%">


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


## 🧩 Attack Workflow Overview

The following table describes each attack phase documented in this repository, along with its corresponding file.

| Step | File | Description |
|------|------|-------------|
| 1️⃣ | [01_Attack_Reconnaissance.md](01_Attack_Reconnaissance.md) | Information gathering — scanning, enumeration, and footprinting of target systems and services. |
| 2️⃣ | [02_Attack_BruteForce.md](02_Attack_BruteForce.md) | Attempts unauthorized access via brute-force and credential-stuffing techniques. |
| 3️⃣ | [03_Attack_Privilege_Escalation.md](03_Attack_Privilege_Escalation.md) | Techniques and exploits used to gain elevated privileges or administrative access. |
| 4️⃣ | [04_Attack_Defense_Evasion_&_Persistence.md](04_Attack_Defense_Evasion_&_Persistence.md) | Methods to avoid detection and maintain persistent access on compromised systems. |
| 5️⃣ | [05_Attack_Cron_Persistence.md](05_Attack_Cron_Persistence.md) | Persistence through malicious cron jobs or scheduled task manipulation. |
| 6️⃣ | [06_Attack_SQL_Injection-Unauthenticated_Bypass.md](06_Attack_SQL_Injection-Unauthenticated_Bypass.md) | Demonstrates SQL Injection attacks, including unauthenticated login bypass techniques. |
| 7️⃣ | [07_Attack_Cross-Site_Scripting_(XSS).md](07_Attack_Cross-Site_Scripting_(XSS).md) | Exploitation of XSS vulnerabilities to inject malicious client-side scripts. |
| 8️⃣ | [08_Attack_Command_Injection.md](08_Attack_Command_Injection.md) | Command injection attacks that allow arbitrary command execution on the host system. |

---

## ⚙️ Usage

Each Markdown file provides:
- Step-by-step attack execution methodology  
- Tools and payloads used  
- Indicators of compromise (IoCs)  
- Possible mitigations and defenses


## ⚙️ Additional Setup & Cleanup Files


| File | Description |
|------|-------------|
| [09_Setup_LAMP.md](09_Setup_LAMP.md) | **LAMP Environment Setup** — installation and configuration of Linux, Apache, MySQL/MariaDB, and PHP. Includes prerequisites, installation commands, virtual host setup, and basic security hardening. |
| [10_Splunk_Cleanup.md](10_Splunk_Cleanup.md) | **Splunk Cleanup & Maintenance** — instructions to safely clean indexes, remove old data, adjust retention settings, and perform Splunk maintenance tasks. |
