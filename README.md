# 🛡️ Unauthorized Access Attempt – Splunk Brute Force Incident  
### Windows Server 2022 Targeted by Ubuntu Host Using Nmap & Hydra

![MITRE T1110](https://img.shields.io/badge/ATT%26CK-T1110%20Brute%20Force-red)
![MITRE T1046](https://img.shields.io/badge/ATT%26CK-T1046%20Network%20Scanning-blue)
![MITRE T1078](https://img.shields.io/badge/ATT%26CK-T1078%20Valid%20Accounts%20(attempted)-orange)
![Splunk](https://img.shields.io/badge/SIEM-Splunk%20Enterprise-green)
![Windows](https://img.shields.io/badge/Target-Windows%20Server%202022-lightgrey)
![Ubuntu](https://img.shields.io/badge/Source-Ubuntu%2024.04-critical)

---

## 📌 Executive Summary  
On **12/29/2025 time**, suspicious authentication and reconnaissance activity was detected targeting a **Windows Server 2022** system (**192.168.84.135**).  
Analysis within Splunk revealed:

- Multiple **Nmap scans** from **192.168.84.134 (Ubuntu)**  
- Repeated **Hydra brute-force attempts** against the `vmw-lab` account  
- Numerous **failed logon events (EventCode 4625)**  
- **No successful authentication** or lateral movement  
- Immediate isolation of the Ubuntu host  

The attack was unsuccessful, but the behavior indicated a compromised or malicious user on the Ubuntu system.

---

## 🧭 Environment Overview

| Component | IP Address | Role |
|----------|------------|------|
| Windows Server 2022 | **192.168.84.135** | Target system (victim) |
| Ubuntu 24.04 | **192.168.84.134** | Source of brute force & scanning |
| Windows 11 Pro | N/A | Splunk Search Head + Indexer |
| Splunk UF | N/A | Installed on Windows Server & Ubuntu |
| Telemetry | N/A | Windows Security Logs + Sysmon |

---

## 🚨 Incident Summary

| Category | Details |
|----------|---------|
| **Type** | Unauthorized access attempt + Reconnaissance |
| **Source** | Ubuntu host (192.168.84.134) |
| **Target** | Windows Server 2022 (192.168.84.135) |
| **Attack Tools** | Nmap, Hydra |
| **Targeted User** | `vmw-lab` |
| **Outcome** | No successful logons |

---

## 🔎 Indicators of Attack  

### ✔ Brute Force Indicators  
- High volume of **EventCode 4625** failed login attempts  
- Consistent targeting of `vmw-lab`  
- Attempts from a single IP (192.168.84.134)  
- No EventCode **4624** (successful logon)  

### ✔ Recon Indicators (Nmap)  
- Sequential connection attempts across common ports  
- Sysmon Event ID 3 logs showing rapid port hits  

---

## 🗂 Log Evidence (Screenshots)

Add your screenshots to `/screenshots/` and embed them below.

### 🔹 Splunk – Failed Logon Summary  
`/screenshots/splunk_4625_summary.png`  
![Failed Logon Summary](screenshots/splunk_4625_summary.png)

### 🔹 Splunk – Timechart of Brute Force Attempts  
`/screenshots/splunk_timechart.png`  
![Brute Force Timechart](screenshots/splunk_timechart.png)

### 🔹 Splunk – Recon Activity (Nmap)  
`/screenshots/splunk_nmap.png`  
![Nmap Recon](screenshots/splunk_nmap.png)

### 🔹 Windows Server – Event Viewer 4625  
`/screenshots/eventviewer_4625.png`  
![Event 4625](screenshots/eventviewer_4625.png)

### 🔹 Ubuntu – Hydra Output  
`/screenshots/hydra_output.png`  
![Hydra Output](screenshots/hydra_output.png)

### 🔹 Ubuntu – Nmap Scan Output  
`/screenshots/nmap_output.png`  
![Nmap Output](screenshots/nmap_output.png)

---

## 🧪 Splunk Detection Queries

### 🔹 Failed Authentication Summary  
```spl
index=main host="WIN-SERVER" EventCode=4625
| stats count by Account_Name, IpAddress, FailureReason
| sort - count
