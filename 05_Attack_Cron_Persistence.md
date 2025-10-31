# 👻 Cron Persistence Objective

The primary goal is to ensure the attacker maintains access to the victim machine, even after system reboots or service restarts, completing the Persistence (TA0003) phase of the attack lifecycle. This is achieved by maliciously leveraging the system's Cron scheduling utility (T1053.003). The attacker appends a command to the system's crontab to execute a hidden malicious script at specified intervals (e.g., every minute), allowing for automated code execution and continued control of the system without needing to exploit it again.


-----

## 🛡️ Difference from Previous Attacks

Your previous attacks likely focused on **gaining initial entry** and then **elevating privileges**. This attack assumes those are *already complete* and is about maintaining a long-term presence.

| Aspect | Initial Access/Privilege Escalation Attacks (e.g., Attack 3) | Cron Persistence Attack (This Attack) |
| :--- | :--- | :--- |
| **MITRE Tactic** | Initial Access (TA0001), Privilege Escalation (TA0004) | **Persistence (TA0003)**, Defense Evasion (TA0005) |
| **Primary Goal** | Get a shell on the system; get root access. | **Maintain access** indefinitely, surviving reboots and logouts. |
| **State Required** | Only a foothold or an unprivileged user. | **Root access (`sudo` or a root shell) is necessary** to modify system-level crontabs or place files in system directories like `/usr/local/bin/`. |
| **Focus** | Exploiting a vulnerability or misconfiguration. | Establishing a **backdoor mechanism** for future, automated re-entry. |
| **Effect** | Temporary gain; access can be lost if the machine reboots. | **Permanent and automated** execution; access is maintained. |

**In short:** You first broke into the house (Initial Access), then you got the master key (Privilege Escalation), and now you're installing a secret, self-locking door that opens every time the system starts (Persistence).

-----

## 🎯 Attack Objective & MITRE Mapping

| Aspect | Description | MITRE ATT\&CK Tactic & Technique |
| :--- | :--- | :--- |
| **Objective** | To establish a reliable, automated way to execute code on the victim machine at specified intervals, ensuring access is maintained after reboots. | **Persistence (TA0003)** |
| **Method** | Utilizing the Cron scheduling utility to execute a persistent script. | **Scheduled Task/Job (T1053.003 - Cron)** |
| **Status** | **SUCCESSFUL** - Persistence established and detection confirmed via Splunk. | |

-----

## 🔴 Part 1: Red Team Execution (Establishing Persistence)

**Starting Point:** Connection to the **Target Linux Machine** with `sudo` or root privileges.

The following commands create a harmless script (`persist.sh`) that logs a timestamp, mimicking how a malicious payload would be set up for long-term control.

### Step 1: Create Staging Directory

```bash
sudo mkdir -p /usr/local/bin/lab
```

*Purpose: Creates a non-standard location to stage the script, aiding in Defense Evasion.*

### Step 2: Create and Populate the Persistence Script

This creates the file and writes the script's content.

```bash
# Part A: Create the file and add the shebang
echo '#!/bin/bash' | sudo tee /usr/local/bin/lab/persist.sh

# Part B: Append the benign payload (logs the date)
echo 'date >> /var/log/lab_persist.log' | sudo tee -a /usr/local/bin/lab/persist.sh
```

### Step 3: Make Script Executable and Verify

```bash
# Grant execution permission
sudo chmod +x /usr/local/bin/lab/persist.sh

# Verify file content (optional check)
cat /usr/local/bin/lab/persist.sh
```

### Step 4: Establish the Cron Job (Core Attack Command)

This command reads the existing crontab, appends the new entry to run the script every minute (`* * * * *`), and saves the changes as the root user.

```bash
(sudo crontab -l 2>/dev/null; echo "* * * * * /usr/local/bin/lab/persist.sh") | sudo crontab -

# Verification of Crontab Entry:
sudo crontab -l
```

### Step 5: Final Execution Verification

```bash
sleep 65 && tail /var/log/lab_persist.log
```

*Result: The output showed multiple timestamps, confirming the script is executing every minute.*

-----

## 🔎 Part 2: Blue Team Detection (Splunk Analysis)

The detection strategy focused on flagging the modification to the system scheduler and the continuous execution of the anomalous script.

### 1\. Detection of Crontab Modification (Initial Alert)

This query attempts to find the log entry generated when the `crontab -` command modified the root user's schedule. This is the **highest-fidelity alert**.

| Detection Query | Purpose |
| :--- | :--- |
| `index=* (CRON OR crontab) "INSTALLING new crontab"` | Catches the specific log message detailing the configuration change. |
| `index=* (CRON OR crontab) "CMD (crontab"` | Broader search to catch the execution of the `crontab -` command itself. |

### 2\. Detection of Anomalous Script Execution (Confirmed Persistence)

This query confirmed the attack by successfully flagging the periodic execution of the script from the custom, non-standard directory.

| Detection Query | Purpose |
| :--- | :--- |
| `index=* (CRON OR crontab) AND cmd "/usr/local/bin/lab/persist.sh"` | Detects the system's logging of the CRON daemon executing the specific persistence script. |

### 3\. Splunk Dashboard Visualization

The detection was visualized using two panels:

| Panel Type | Query | Insight Provided |
| :--- | :--- | :--- |
| **Events Table** | `index=* (CRON OR crontab) AND cmd "/usr/local/bin/lab/persist.sh"\| table _time, host, source, cmd` | Provides definitive proof of the script's path and execution time. |
| **Timechart** | `index=* (CRON OR crontab) AND cmd "/usr/local/bin/lab/persist.sh"\|timechart count` | Confirms the nature of the attack by showing a clear, regular spike in execution (one event per minute). |

-----

## 🧹 Part 3: Remediation (Clean-Up)

To properly clean the victim system and complete the project narrative, the persistence mechanism must be removed.

### Step 1: Remove the Cron Job Entry

The safest way is to edit the root crontab and remove the line `* * * * * /usr/local/bin/lab/persist.sh`.

```bash
sudo crontab -e
```

*(In the editor, delete the line and save the file.)*

### Step 2: Delete the Persistence Files

Remove the script and the staging directory.

```bash
sudo rm /usr/local/bin/lab/persist.sh
sudo rm -r /usr/local/bin/lab
sudo rm /var/log/lab_persist.log
```

-----
