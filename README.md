# MYDFIR-full-domain-compromise-investigation
Realistic DFIR investigation simulating a full domain compromise attack, including phishing, lateral movement, credential dumping, and data exfiltration.
# 🔴 Full Domain Compromise Incident Report

## 📌 Overview

- **Date:** March 20, 2026  
- **Severity:** Critical  
- **Status:** Active  
- **Analyst:** Abdoulaye  

---

# 🧾 Executive Summary

On January 30, 2026, a user opened a phishing email and downloaded a malicious file.

This action allowed an attacker to:
- Access the internal network
- Steal credentials
- Move across systems
- Take control of the Domain Controller
- Steal sensitive company data

⚠️ The entire domain was compromised in less than **2 hours**.


---

# 🧭 Incident Overview (5W)

## 👤 Who (Threat Actor & Compromised Accounts)

**Threat Actor Details:**

| Indicator | Value |
|-----------|-------|
| Exfil IP | 66.203.125.15 |
| Email / Password | jwilson.vhr@proton.me / Summer2024! |
| Exfil Service | MEGA |
| C2 Domain | cdn.cloud-endpoint.net |
| C2 IP | 104.21.30.237 |

**Compromised Devices:**

| Device | Role | Local IP |
|--------|------|---------|
| EC2AMAZ-B9GHHO6 | Workstation | 10.1.173.145 |
| EC2AMAZ-16V3AU4 | File Server | 10.1.57.66 |
| EC2AMAZ-EEU3IA2 | Domain Controller | 10.1.160.76 |

**Abused Accounts:**

- lmartin  
- Administrator  
- SYSTEM  
- EC2AMAZ-EEU3IA2$  

---

## ❓ What (What Happened)

- User opened a **phishing email** and downloaded a malicious file  
- Malware ran (`rundll32.exe review.dll`) and loaded **Sliver beacon**  
- Attacker accessed the network without permission  
- Internal reconnaissance done (users, accounts, systems)  
- Secondary malware (`update.exe`) uploaded  
- **Admin privileges gained** via UAC bypass  
- Credentials stolen (LSASS memory, NTDS.dit)  
- Moved to file server and Domain Controller  
- Persistence via **scheduled tasks** and **AnyDesk**  
- Sensitive files sent to MEGA  

---

## ⏱️ When (Timeline Summary)

### Workstation (EC2AMAZ-B9GHHO6)
- **09:22** – Malicious file downloaded  
- **09:27** – Malware executed  
- **09:34–09:36** – Reconnaissance and payload upload  
- **09:37** – Scheduled task created  
- **09:38** – UAC bypass  
- **10:14** – Lateral movement  
- **10:19** – AnyDesk installed  
- **10:54** – Firewall changed for SMB  

### File Server (EC2AMAZ-16V3AU4)
- **10:07–10:52** – Lateral movement, AnyDesk and update.exe deployed  
- **11:05–11:08** – Sensitive files staged and exfiltrated  

### Domain Controller (EC2AMAZ-EEU3IA2)
- **11:35** – NTDS.dit copied via Volume Shadow Copy  
- **11:38** – Backup account `svc_backup` created  
- **11:47** – Scheduled task created  

---

## 📍 Where (Affected Systems)

- Workstation: EC2AMAZ-B9GHHO6 (User: lmartin)  
- File Server: EC2AMAZ-16V3AU4 (Accounts: SYSTEM, Administrator)  
- Domain Controller: EC2AMAZ-EEU3IA2 (Accounts: SYSTEM, Administrator)  

**Directories used by attacker:**
- `C:\Windows\TEMP\`  
- `C:\Users\Public\`  

**Malware path:**
- `C:\Users\Public\update.exe`  

---

## ❗ Why (Root Cause)

1. User clicked a phishing email  
2. Weak Domain Administrator password  
3. Admin credentials accessible from workstation  
4. Endpoint protection did not block attacks  
5. Internal network allowed fast lateral movement  

---

# 🖥️ Affected Systems

| Hostname | Role | IP |
|----------|------|----|
| EC2AMAZ-B9GHHO6 | Workstation | 10.1.173.145 |
| EC2AMAZ-16V3AU4 | File Server | 10.1.57.66 |
| EC2AMAZ-EEU3IA2 | Domain Controller | 10.1.160.76 |

---

# 👤 Threat Infrastructure

- **C2 Domain:** `cdn.cloud-endpoint.net`
- **C2 IP:** `104.21.30.237`
- **Exfiltration IP:** `66.203.125.15`
- **Cloud Storage:** MEGA
- **Email:** jwilson.vhr@proton.me

---

# ⚙️ Attack Flow 

## 1. Initial Access
User downloaded a malicious ISO file from a phishing email.

(images/initial-access.png) <img width="1602" height="487" alt="image" src="https://github.com/user-attachments/assets/ac1c6d5c-be75-4b63-a682-eda11751bb0b" />

KQL Query 


---

## 2. Execution
Malware executed using `rundll32.exe`.

<img width="1516" height="453" alt="image" src="https://github.com/user-attachments/assets/e5f12ed2-9cba-486b-a64b-3333cbe6bb51" />

<img width="1516" height="453" alt="image" src="https://github.com/user-attachments/assets/6b91f050-b608-430b-b63c-1fcbe5f45c4e" />



---

## 3. Persistence
- Scheduled task created (`WindowsUpdate`)
- AnyDesk installed

<img width="1640" height="347" alt="image" src="https://github.com/user-attachments/assets/e6f2f7df-f4a9-496d-937c-06bfbdcfa5d7" />

<img width="1643" height="314" alt="image" src="https://github.com/user-attachments/assets/083da342-a956-48f2-a34b-d18914259bb7" />

<img width="1658" height="251" alt="image" src="https://github.com/user-attachments/assets/27f1d094-d7dc-45cb-92ca-9948a2d22060" />

<img width="1658" height="251" alt="image" src="https://github.com/user-attachments/assets/50b90902-3bec-4f49-b04a-bb7658619b32" />





---

## 4. Privilege Escalation
- UAC bypass
- SYSTEM access gained

![Privilege Escalation](images/privilege-escalation.png)

---

## 5. Credential Access
- Passwords dumped from LSASS
- Domain Admin credentials stolen

![Credential Access](images/credential-access.png)

---

## 6. Lateral Movement
- Moved across systems using SMB

![Lateral Movement](images/lateral-movement.png)

---

## 7. Data Collection
Sensitive files gathered and prepared

![Collection](images/collection.png)

---

## 8. Command & Control
Connection established to attacker server

![C2](images/c2.png)

---

## 9. Data Exfiltration
Data uploaded to MEGA using `rclone`

![Exfiltration](images/exfiltration.png)


---

# 💥 Impact

- Full domain compromise  
- All credentials exposed  
- Sensitive data stolen  
- Persistent attacker access  
- Multiple systems infected  

---

# 🧩 MITRE ATT&CK Mapping

| Tactic | Technique | ID | Description |
|--------|----------|----|------------|
| Initial Access | Phishing Attachment | T1566.001 | User opened malicious ISO file |
| Execution | User Execution | T1204.002 | User launched infected file |
| Execution | Command Execution | T1059 | rundll32 used to run DLL |
| Defense Evasion | Obfuscated Files | T1027 | Malware hidden in ISO |
| Defense Evasion | Signed Binary Proxy | T1218.011 | rundll32 abused |
| Persistence | Scheduled Task | T1053.005 | WindowsUpdate task created |
| Persistence | Remote Access Software | T1219 | AnyDesk installed |
| Privilege Escalation | UAC Bypass | T1548.002 | fodhelper.exe used |
| Privilege Escalation | Exploit for Privilege | T1068 | Print Spooler abuse |
| Credential Access | LSASS Dumping | T1003.001 | Passwords extracted from memory |
| Credential Access | NTDS Dumping | T1003.003 | Domain database stolen |
| Discovery | System Discovery | T1082 | System info collected |
| Discovery | Account Discovery | T1033 | User enumeration |
| Lateral Movement | SMB / Admin Shares | T1021.002 | Movement across systems |
| Collection | Data Staging | T1560 | Files prepared |
| Command & Control | Web Protocols | T1071.001 | C2 communication over HTTPS |
| Exfiltration | Cloud Storage | T1567.002 | Data sent to MEGA |
| Defense Evasion | Log Clearing | T1070 | Evidence removed |

---

# 🛠️ Recommendations

## Immediate Actions
- Reset all passwords  
- Remove `svc_backup` account  
- Isolate infected systems  
- Remove malware and AnyDesk  
- Block malicious IPs  

## Long-Term Improvements
- Enable MFA  
- Improve phishing training  
- Network segmentation  
- Deploy EDR  
- Regular security audits  

---

# 📚 Key Lessons

- One click can compromise everything  
- Protect admin accounts strictly  
- Use strong passwords  
- Limit internal access  
- Monitor behavior, not just signatures  

---

# 🔍 Indicators of Compromise (IOCs)

## Files
- `update.exe`
- `rclone.exe`
- `EmberForge_Review.iso`

## Accounts
- `svc_backup`

## Network
- `66.203.125.15`
- `104.21.30.237`

---

# 🧠 Final Note

This incident shows how quickly attackers can move:

➡️ One phishing email  
➡️ One compromised machine  
➡️ Full domain takeover  

---
