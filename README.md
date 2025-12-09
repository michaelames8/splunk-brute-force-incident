# 🛡️ Unauthorized Access – Brute Force & Reconnaissance Incident  
### SIEM: Windows 11 Pro VM (Win11 – 192.168.84.131) running Splunk Enterprise
### Source: Ubuntu Host VM (ubuntulab – 192.168.84.134)
### Target: Windows Server 2022 VM (WIN-1BRV561EKE1 – 192.168.84.135)  


---

# 🎖️ MITRE ATT&CK Techniques  
[![T1110 – Brute Force](https://img.shields.io/badge/MITRE-T1110_Brute_Force-red?logo=target)](https://attack.mitre.org/techniques/T1110/) &nbsp;&nbsp;&nbsp;[![T1046 – Network Scanning](https://img.shields.io/badge/MITRE-T1046_Network_Scanning-blue?logo=target)](https://attack.mitre.org/techniques/T1046/) &nbsp;&nbsp;&nbsp;[![T1078 – Valid Accounts (Attempted)](https://img.shields.io/badge/MITRE-T1078_Valid_Accounts-orange?logo=target)](https://attack.mitre.org/techniques/T1078/)

---

# 💻 System & Tools
![Windows Server 2022](https://img.shields.io/badge/Windows_Server_2022-0078D6?logo=windows&logoColor=white) &nbsp;&nbsp;&nbsp;![Windows 11 Pro](https://img.shields.io/badge/Windows_11_Pro-0078D6?logo=windows11&logoColor=white) &nbsp;&nbsp;&nbsp;![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu_24.04-E95420?logo=ubuntu&logoColor=white) &nbsp;&nbsp;&nbsp;![Splunk Enterprise](https://img.shields.io/badge/Splunk_Enterprise-000000?logo=splunk&logoColor=white) &nbsp;&nbsp;&nbsp;![Hydra](https://img.shields.io/badge/Hydra-4B275F?logo=hackthebox&logoColor=white) &nbsp;&nbsp;&nbsp;![Nmap](https://img.shields.io/badge/Nmap-215732?logo=linux&logoColor=white)  

---

# 📌 Executive Summary  
On **11/29/2025 at 6:15 PM EST**, multiple failed Remote Desktop Protocol (RDP) authentication attempts were detected on the Windows Server. All attempts originated from a single Ubuntu host within the network. After the initial detection, the Ubuntu host was immediately isolated from the network pending an investigation.
Logs analysis using Splunk Enterprise revealed:

- Network reconnaissance originating from the Ubuntu host using **Nmap**
- Brute force login attempts using **Hydra**
- Repeated login failures by user **vmw-lab** from the Ubuntu computer 
- No successful logons, lateral movement, or data access  

This report documents the findings, timeline, SPL queries, and remediation actions.

---

# 🧭 Environment Overview

| Hostname | IP Address | Operating System | Role |
|----------|------------|------------------|------|
| **Win11** | 192.168.84.131 | Windows 11 Pro | Splunk Search Head + Indexer |
| **WIN-1BRV561EKE1** | 192.168.84.135 | Windows Server 2022 | Target |
| **ubuntulab** | 192.168.84.134 | Ubuntu 24.04 | Source of attack |

---

# 🧪 Investigation
The investigation focused on determining whether the failed logon activity represented unauthorized access attempts, the attack method, and whether credentials were compromised. Splunk searches filtered by IP address revealed that only one system was the target of the attack and all attempts to access the Windows Server originated from one machine, the Ubuntu host (192.168.84.134), ruling out lateral movement within the network.

##SPL Queries Used During Investigation  

### **Failed Authentication Summary**
```spl
index=main host="WIN-1BRV561EKE1" "Failed password" OR "authentication failure" OR EventCode=4625
| stats count 

Time Range: Last 24 hours
```
### **Use of Hydra**
```spl
index=main host="ubuntulab" "hydra"
| sort by _time asc

Time Range: Last 7 days
```
### **Use of Nmap**
```spl
index=main host="ubuntulab" COMMAND="/usr/bin/nmap"
| sort by _time asc

Time Range: Last 7 days 
```

---

# 🧨 Indicators of Attack  

### ✔ Unauthorized Network Scanning  
- Nmap reconnaissance from `ubuntulab (192.168.84.134)`  
- Sequential port probing detected via Sysmon Event ID 3  

### ✔ Brute Force Authentication Attempts  
- High volume of **EventCode 4625** failures  
- Attempts targeted host **WIN-1BRV561EKE1**  
- No EventCode **4624** (successful login)  

### ✔ No Evidence of:  
- An Established Session
- Lateral movement  
- Privilege escalation  
- Persistence mechanisms  

---

# 🔍 Log Evidence & Screenshots  

### 🔹 Failed Logon Dashboard  
![Failed Logon Dashboard](https://github.com/michaelames8/splunk-brute-force-incident/blob/52174e74bda6865e79be97021c079e71d00d385d/screenshots/Dashboard.png)

### 🔹 Brute Force Attempts  
![Brute Force Attempts](https://github.com/michaelames8/splunk-brute-force-incident/blob/ecfc7195b64cc4e7c672ead9d91e9e9216e9b497/screenshots/failed%20logon%20search.png)

![Source of Attack](https://github.com/michaelames8/splunk-brute-force-incident/blob/c9305186c2e6a9d0061749da94d8650782cc32fd/screenshots/dedup%20source%20IP.png)

![Machine Interactions](https://github.com/michaelames8/splunk-brute-force-incident/blob/c9305186c2e6a9d0061749da94d8650782cc32fd/screenshots/Interactions%20between%20machines.png)

### 🔹 Hydra Use 
![Hydra Use](https://github.com/michaelames8/splunk-brute-force-incident/blob/1af5d45f3d637c0371db4531385937bb522a47ff/screenshots/hydra%20use.png)

### 🔹 Nmap Recon Activity  
![Nmap Recon Activity](https://github.com/michaelames8/splunk-brute-force-incident/blob/6a1bb626b4e95454351dc03414de90663ca8dc9a/screenshots/nmap%20use.png)

---

# 🛠️ Recommended Actions

- Rename the default Administrator account and enforce a privileged user account policy.
- Remove the "labuser" account from the Windows Server because it unnecessarily adds a point of entry for an attacker.
- Enforce an account lockout policy that locks an account for more than 5 incorrect passwords within a 24-hour period.
- For enhanced protection, move to a phishing-resistant hardware authentication method (e.g., YubiKey).
- Assess the Ubuntu "vmw-lab" user for an Insider Threat Risk.
- Disable RDP access to the Windows Server.
- Conduct regular audits of privileged accounts to ensure the activity is consistent with the AUP and the normal behavior of the user.  
