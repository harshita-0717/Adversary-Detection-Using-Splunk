## 🔑 Attack Phase 2: SSH Brute Force (Initial Access)

### 🎯 Objective

This phase simulates an attacker gaining **Initial Access** by systematically trying numerous username and password combinations against the running **SSH service** on the target Ubuntu VM. This exercise generates crucial security logs for our detection analysis.

### 📜 MITRE ATT\&CK Mapping

| Tactic (Attacker's Goal) | Technique ID | Technique Name | Description |
| :--- | :--- | :--- | :--- |
| **Credential Access** (TA0006) | T1110 | Brute Force | Automated tool (Hydra) used to guess credentials against the SSH service. |

-----

### 💻 System Details

| Detail | Value |
| :--- | :--- |
| **Attacker IP (Kali)** | `192.168.56.101` |
| **Target IP (Ubuntu)** | `192.168.56.103` |
| **Tool** | Hydra |


#### 1\. Password Wordlist Creation

A file named `ssh_passwords.txt` was created to feed passwords to the Hydra tool.

```bash
# Command to create the wordlist
nano ssh_passwords.txt
```

**Wordlist Content:**

```plaintext
password123
kali
ubuntu
admin
testpass
wrong
hk17
12345
hydra
# [One of these passwords was the correct one to stop the attack]
```

#### 2\. Hydra Execution

The Hydra command tested every password against the target user (`harshu`) on the Ubuntu VM.

```bash
# SYNTAX: hydra -l [username] -P [password_file] [protocol]://[target_ip]
hydra -l harshu -P ssh_passwords.txt ssh://192.168.56.103
```

**Expected Result (Hydra Success):**
The tool generated many failed attempts, followed by the successful credential:
`[22][ssh] host: 192.168.56.103 login: harshu password: [found_password]`

| Detail | Value |
| :--- | :--- |
| **Start Time** | `2025-10-26 19:48:43` |
| **End Time** | `2025-10-26 19:48:48` |
| **Tool** | Hydra |

-----


### 🕵️ Log Analysis and Detection (Blue Team - Splunk)

The many failed logins were instantly written to the `/var/log/auth.log` file, which Splunk was monitoring.
Start the splunk web interface - `http://localhost:8000`

#### 1\. Raw Log Verification

A simple Splunk search was run to confirm the logs were being ingested and to isolate the raw events.

```splunk
index=os_logs sourcetype=linux_secure earliest=-5m
```

**Log Confirmation:** The raw logs on the Ubuntu system displayed numerous entries like: `Failed password for harshu from 192.168.56.101`.

-----

#### 2\. Detection Query: Failed Logins

Filter those logs to find the specific login failure messages from the attacking IP:

```splunk
index=os_logs sourcetype=linux_secure "Failed password" src_ip="192.168.56.103"
```

-----

#### 3\. Detection Query: Brute Force Threshold

To detect the attack reliably, an **analytical SPL query** was used to aggregate the failed attempts, moving beyond individual events to identify the malicious *pattern* of a brute force attack (T1110).

```splunk
index=os_logs sourcetype=linux_secure "Failed password" 
| stats count as failure_count by user, src_ip 
| where failure_count > 5
| table _time, user, src_ip, failure_count 
| sort -failure_count
```

  * **Logic:** The query counts all failed passwords per `src_ip` and `user`. The **`where failure_count > 5`** filter acts as the detection threshold, flagging only high-volume suspicious activity.

#### 4\. Detection Result

The query successfully highlighted the Kali IP with a high count of login failures, confirming the successful detection of the Brute Force attack.

**Detection Summary:**

  * **Attacker IP:** `192.168.56.101`
  * **Technique:** T1110 Brute Force
  * **Result:** Clearly detected due to high `failure_count` exceeding the threshold.

-----

-----

### 💡 Next Steps in Incident Handling

After detecting this incident, the SOC analyst's immediate actions would be:

1.  **Block:** Immediately block the source IP (`192.168.56.101`) at the firewall level (simulated).
2.  **Report:** Create an incident report, citing the MITRE ATT\&CK technique and the successful detection query.
3.  **Remediate:** Implement stronger security controls (e.g., enable Fail2ban or use SSH Key-Based Authentication to prevent future brute force attacks).

-----
