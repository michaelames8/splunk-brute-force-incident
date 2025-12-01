# 🛡️ Unauthorized Access – Brute Force & Reconnaissance Incident  
### Target: Windows Server 2022 (WIN-1BRV561EKE1 – 192.168.84.135)  
### Source: Ubuntu Host (ubuntulab – 192.168.84.134)  
### SIEM: Windows 11 Pro (Win11 – 192.168.84.131) running Splunk Enterprise

---

# 🎖️ MITRE ATT&CK Techniques  
[![T1110 – Brute Force](https://img.shields.io/badge/MITRE-T1110_Brute_Force-red?logo=target)](https://attack.mitre.org/techniques/T1110/)  
[![T1046 – Network Scanning](https://img.shields.io/badge/MITRE-T1046_Network_Scanning-blue?logo=target)](https://attack.mitre.org/techniques/T1046/)  
[![T1078 – Valid Accounts (Attempted)](https://img.shields.io/badge/MITRE-T1078_Valid_Accounts-orange?logo=target)](https://attack.mitre.org/techniques/T1078/)

---

# 💻 System & Tools
![Windows Server 2022](https://img.shields.io/badge/Windows_Server_2022-0078D6?logo=windows&logoColor=white)  
![Windows 11 Pro](https://img.shields.io/badge/Windows_11_Pro-0078D6?logo=windows11&logoColor=white)  
![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu_24.04-E95420?logo=ubuntu&logoColor=white)  
![Splunk Enterprise](https://img.shields.io/badge/Splunk_Enterprise-000000?logo=splunk&logoColor=white)  
![Hydra](https://img.shields.io/badge/Hydra-4B275F?logo=hackthebox&logoColor=white)  
![Nmap](https://img.shields.io/badge/Nmap-215732?logo=linux&logoColor=white)  

---

# 📌 Executive Summary  
On **11/29/2025 time**, suspicious authentication and reconnaissance activity was detected on the Windows Server 2022 system (**WIN-1BRV561EKE1 – 192.168.84.135**).  
Logs forwarded into Splunk Enterprise (hosted on the Windows 11 Pro VM **Win11 – 192.168.84.131**) revealed:

- Network reconnaissance originating from **ubuntulab – 192.168.84.134** using **Nmap**
- Brute force login attempts using **Hydra**
- Repeated login failures against user **vmw-lab**  
- No successful logons or lateral movement  
- The Ubuntu host was immediately isolated from the network  
- The targeted user password was reset  

This report documents the findings, timeline, SPL queries, and remediation actions.

---

# 🧭 Environment Overview

| Hostname | IP Address | Operating System | Role |
|----------|------------|------------------|------|
| **Win11** | 192.168.84.131 | Windows 11 Pro | Splunk Search Head + Indexer |
| **WIN-1BRV561EKE1** | 192.168.84.135 | Windows Server 2022 | Target (Victim) |
| **ubuntulab** | 192.168.84.134 | Ubuntu 24.04 | Source of attack (compromised) |

Telemetry sources:

- **Windows Security Logs** (EventCode 4625)
- **Sysmon Logs** (network & process telemetry)
- **Splunk Universal Forwarder** on Windows Server + Ubuntu

---

# 🧨 Indicators of Attack  

### ✔ Unauthorized Network Scanning  
- Nmap reconnaissance from `ubuntulab (192.168.84.134)`  
- Sequential port probing detected via Sysmon Event ID 3  

### ✔ Brute Force Authentication Attempts  
- High volume of **EventCode 4625** failures  
- Attempts targeted user **vmw-lab**  
- No EventCode **4624** (successful login)  

### ✔ No Evidence of:  
- Session establishment  
- Lateral movement  
- Privilege escalation  
- Persistence mechanisms  

---

# 🔍 Log Evidence & Screenshots  

Add your images into `/screenshots/` and they will display here.

### 🔹 Splunk: Failed Logon Summary  
![Failed Logon Summary](screenshots/splunk_4625_summary.png)

### 🔹 Splunk: Timechart of Brute Force Attempts  
![Brute Force Timechart](screenshots/splunk_timechart.png)

### 🔹 Splunk: Recon Activity (Nmap)  
![Nmap Recon Activity](screenshots/splunk_nmap.png)

### 🔹 Windows Event Viewer (4625)  
![Event 4625](screenshots/eventviewer_4625.png)

### 🔹 Hydra Output (Ubuntu)  
![Hydra Output](screenshots/hydra_output.png)

### 🔹 Nmap Scan Output  
![Nmap Output](screenshots/nmap_output.png)

---

# 🧪 SPL Queries Used During Investigation  

### **Failed Authentication Summary**
```spl
index=main host="WIN-1BRV561EKE1" EventCode=4625
| stats count by Account_Name, IpAddress, FailureReason
| sort - count
