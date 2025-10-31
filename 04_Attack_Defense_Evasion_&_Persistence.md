# 👻 Attack Phase 4: Defense Evasion & Persistence

This final attack phase demonstrates the attacker's ability to maintain long-term access and remove security measures, which is crucial for full system control and continued presence.

## 📜 MITRE ATT\&CK Mapping

| Tactic | Technique ID | Technique Name |
| :--- | :--- | :--- |
| **Defense Evasion** (TA0005) | T1562.004 | Impair Defenses: Disable System Firewall (UFW) |
| **Persistence** (TA0003) | T1136.001 | Create Account: Local Account (Backdoor) |

-----

## 💻 Part A: Red Team Execution (via Kali SSH as `root`)

All commands were run from the **Kali Linux terminal** inside the root shell (`root@...#`) on the Ubuntu VM.

| Action | Command Executed | Purpose and Impact |
| :--- | :--- | :--- |
| **Defense Evasion** | `ufw disable` | Disables the **Uncomplicated Firewall (UFW)** on the target system. This removes local network security and simplifies future external connections (T1562.004). |
| **Persistence (User Create)** | `useradd -m -s /bin/bash sadmin` | Creates a new, unauthorized, non-standard user account (`sadmin`) for guaranteed future access. |
| **Persistence (Privilege)** | `usermod -aG sudo sadmin` | **Adds the `sadmin` user to the `sudo` group.** This grants the persistent account full **root-level privileges** (T1136.001). |

-----

## 🛡️ Part B: Blue Team Detection and Log Analysis

This phase tested the Blue Team's ability to monitor both **authentication** (Persistence) and **system configuration** (Defense Evasion) changes.

### 1\. Detection: Unauthorized Account Creation (Persistence)

The Linux authentication logs (`auth.log`) robustly tracked the user creation and privilege changes, providing clear evidence of **Persistence (T1136.001)**.

#### **Evidence Found**

  * **Success:** Logs confirmed the `useradd` and `usermod` commands executed by the root user.
  * **Critical Proof:** The log showed the new user being added to the `sudo` group, confirming root-level privilege was granted.

#### **Detection Query (High-Fidelity Persistence)**

Since the default log output was messy, we searched for key keywords to isolate the events:

```splunk
index=os_logs sourcetype=linux_secure "new user" OR "useradd" OR "usermod" 
| table _time, host, event
```

  * **Critical Learning:** The creation of new user accounts, especially those immediately granted high privileges (`usermod -aG sudo`), is a **Tier 1 indicator of compromise (IOC)**.

### 2\. Detection: Firewall Status Change (Defense Evasion)

This detection was challenging because standard system logs often fail to record granular service status changes.

#### **Log Challenge Identified**

  * **Root Cause:** The system's default logging focused on `/var/log/auth.log` but often missed the specific command output for `ufw disable` in `/var/log/syslog`.
  * **Solution/Hardening:** To guarantee detection, the Blue Team identified the need to add a dedicated Splunk input for the firewall's specific log file: **`/var/log/ufw.log`**.

#### **Detection Procedure**

1.  **Blue Team Hardening:** Added a new Splunk input for **`/var/log/ufw.log`** with `sourcetype=ufw`.
2.  **Red Team Re-Execution:** The `ufw disable` command was run again to hit the new log input.

#### **Final Detection Query**

This query targeted the specific log source for the UFW service status change:

```splunk
index=os_logs sourcetype=ufw "disabled" OR "stop"
| table _time, host, event
```

  * **Critical Learning:** This confirms that **effective detection requires closing logging gaps** by creating specialized inputs for specific applications (like UFW or Auditd). The detection of this event confirms the attacker's **Defense Evasion (T1562.004)**.

-----

## 💡 Conclusion

By successfully simulating and detecting the final stages of **Persistence** and **Defense Evasion**, this has demonstrated comprehensive knowledge of the entire **Red Team Kill Chain** and proven the ability to:

1.  **Identify and fix logging gaps** (e.g., adding `auditd` and `ufw.log`).
2.  **Develop high-fidelity detection logic** in a SIEM (Splunk).
3.  **Map complex incidents** back to the MITRE ATT\&CK Framework.

-----

Would you like final guidance on compiling your full project documentation, including the organization and presentation of your final dashboards?
