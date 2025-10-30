## 🔪 Attack 3: Privilege Escalation (The Critical Phase)

**Goal:** After successfully logging in as the low-privileged user `harshu` via SSH, the attacker must gain **root** privileges to control the entire system, install backdoors, and delete logs.

### 📜 MITRE ATT\&CK Mapping

| Tactic (Attacker's Goal) | Technique ID | Technique Name | Description |
| :--- | :--- | :--- | :--- |
| **Privilege Escalation** (TA0004) | T1055 | Process Injection (Proxy Technique) | Gaining higher-level permissions to execute commands as root. |
| **Defense Evasion** (TA0005) | T1070.004 | Indicator Removal: File Deletion | Using commands to clear logs to hide activity. |

-----

### Part A: Red Team Execution (Kali $\rightarrow$ Ubuntu)

You must first **SSH into the Ubuntu VM** using the credentials you found in Attack 2. This is the starting point for all post-exploitation activities.

#### 1\. Initial Access (SSH Login)

1.  **Open a terminal** on your **Kali Linux VM**.

2.  **Log in** to your Ubuntu VM using the compromised credentials:

    ```bash
    ssh harshu@192.168.56.103  # Use your Ubuntu IP
    ```

      * **Explanation:** The attacker establishes a secure, interactive shell session on the target as the regular user `harshu`.

#### 2\. The Privilege Escalation Attempt (The Malicious Command)

A simple, common, yet powerful Privilege Escalation technique is to exploit the **`sudo` configuration**. If a user is part of the `sudo` group but their password is weak, or if the `sudo` configuration is improperly set, the attacker can use a utility that's allowed to run as root.

We will simulate this by immediately attempting to become the root user:

1.  **On the Ubuntu Shell (logged in as `harshu`),** run the following:

    ```bash
    sudo -i
    ```

2.  **Enter the password** for the `harshu` user (the one you brute-forced).

      * **Explanation:** The command **`sudo -i`** asks the system to switch the user to the root account (`-i` means interactive/login shell). If the system accepts the user's password, the shell prompt will change from `harshu@...$` to **`root@...#`**. This confirms you have successfully achieved **Privilege Escalation**.

#### 3\. Defense Evasion (Log Clearing)

An attacker who gains root access will immediately try to destroy evidence.

1.  **On the root shell (`root@...#`),** run the following commands:

    ```bash
    # Clear the shell history of the current session
    history -c && echo > ~/.bash_history

    # Attempt to clear the actual system logs (The crucial log file!)
    echo '' > /var/log/auth.log
    ```

      * **Explanation:** The attacker clears the shell history (T1070.003 - Command History Modification) and then attempts to **wipe the `/var/log/auth.log`** file (T1070.004 - File Deletion). This action generates a spike in file modification events and a massive change in the log file size, which is highly detectable.

-----

### Part B: Blue Team Detection (Splunk Analysis)

Switch immediately to your Splunk Web UI and act as the SOC Analyst.

#### 1\. Detection Query: Log Clearing Attempt (High Priority Alert)

The attacker's attempt to clear the log file is extremely suspicious. We look for the system utility that performs this action (`echo` or similar commands running with root privileges).

```splunk
# Look for root commands attempting to write or modify security log files
index=os_logs sourcetype=linux_secure (user=root OR command=sudo) "/var/log/auth.log"
| table _time, user, command, event
```

  * **Critical Learning:** Detecting an unauthorized user attempting to delete a critical security log file is a **Tier 1, immediate alert** in any SOC.

#### 2\. Detection Query: The Sudo-to-Root Event (Privilege Escalation)

The change in user privileges (when `harshu` ran `sudo -i`) is recorded in the `auth.log`. This is the direct proof of **Privilege Escalation (TA0004)**.

```splunk
# Find the exact moment the user elevated privileges
index=os_logs sourcetype=linux_secure "session opened" "harshu"
| search "uid=0(root)"
| table _time, user, event, src_ip
```

  * **Critical Learning:** The log message will contain **`session opened for user root by (uid=0)`**, indicating the exact time the attacker successfully transitioned to the highest privilege level using the compromised credentials.

-----
