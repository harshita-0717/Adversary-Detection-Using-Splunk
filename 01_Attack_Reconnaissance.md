## 🗺️ Attack Phase 1: Network Reconnaissance (Nmap Scan)

### 🎯 Objective

This initial phase simulates an attacker performing **reconnaissance** to gather information about the target system (Ubuntu VM). The goal is to discover which ports are open and what services (like SSH, HTTP, etc.) are running, which is crucial for planning the next stage of attack.

### 📜 MITRE ATT&CK Mapping

| Tactic (Attacker's Goal)    | Technique ID | Technique Name                       | Description                                                         |
| :-------------------------- | :----------- | :----------------------------------- | :------------------------------------------------------------------ |
| **Reconnaissance** (TA0043) | T1595.002    | Active Scanning: Vulnerability Scans | Directly scanning the target to find running services and versions. |
| **Discovery** (TA0007)      | T1046        | Network Service Scanning             | Using tools to scan ports and identify network services.            |

---

### 💻 Execution Steps (Kali Linux)

The Nmap tool was used from the Kali Linux VM (Attacker IP: `192.168.56.101`) against the Ubuntu VM (Target IP: `192.168.56.103`).

#### 1. Basic SYN Scan for Open Ports

This scan attempts to quickly identify active ports using the SYN-ACK handshake method.

```bash
# Nmap Command: Quick scan of the top 1000 ports
nmap -sS 192.168.56.103
```

#### 2. Service and Version Detection Scan

Once the open port (Port 22 for SSH) was confirmed, a more detailed scan was executed to determine the specific service and its version. This information is vital for finding potential zero-day or known exploits.

```bash
# Nmap Command: Detailed service version detection
nmap -sV 192.168.56.103
```

---

*Screenshot Placeholder: Please add the screenshot showing the Nmap command and its output, confirming Port 22/SSH is open.*

---

### 🔎 Detection Analysis in Splunk (Blue Team Perspective)

The network scan activity generated specific logs related to connection attempts, which were captured by the **`/var/log/syslog`** and indexed in Splunk.

#### 1. Search Query to Find Scan Activity

The SOC Analyst (Blue Team) searched for connection logs originating from the known attacker IP (`192.168.56.101`).

```splunk
index=os_logs src_ip="192.168.56.101" earliest=-5m
| table _time, src_ip, dest_port, event
```

#### 2. Creating a Detection Rule for Port Scanning

A true detection rule focuses on identifying *rapid, successive connections* (a high volume of events) from a single source to multiple ports over a short period.

```splunk
# Detection Logic for Port Scan (T1046)
index=os_logs 
| stats dc(dest_port) as unique_ports_hit, count as total_hits by src_ip 
| where unique_ports_hit > 10 AND total_hits > 50 
| table _time, src_ip, unique_ports_hit, total_hits
```

---

*Screenshot Placeholder: Please add the screenshot showing the results of the second SPL query, which successfully counts the `unique_ports_hit` from the Kali IP.*

---

### 📝 Conclusion

The Nmap activity was successfully executed and logged. By developing an SPL query using `stats dc(dest_port)` and setting a reasonable threshold, we were able to create a signature that reliably detects **Port Scanning (T1046)**, effectively moving from raw log data to actionable threat intelligence.
