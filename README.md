# MYDFIR-full-domain-compromise-investigation
Realistic DFIR investigation simulating a full domain compromise attack, including phishing, lateral movement, credential dumping, and data exfiltration.
# 🔴 Full Domain Compromise Incident Report

## 📌 Overview

- **Date:** March 20, 2026  
- **Severity:** Critical  
- **Status:** Active  
- **Analyst:** Abdoulaye
- **Environment:** Windows domain  
- **Investigation Tool:** Microsoft Defender for Endpoint (MDE) using KQL queries

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

<img width="1602" height="487" alt="image" src="https://github.com/user-attachments/assets/ac1c6d5c-be75-4b63-a682-eda11751bb0b" />

The ZIP file was extracted and likely double-clicked, allowing the attacker to gain access.

<img width="1641" height="452" alt="image" src="https://github.com/user-attachments/assets/a28ed1d0-16dc-42ce-b9f7-df19d5fceeff" />

This query was used to identify files downloaded from web browsers on the compromised device.



---

## 2. Execution
The malicious file executed: D:\review.dll,StartW`
This loaded the Sliver DLL beacon into memory and connected to the attacker C2 at `cdn.cloud-endpoint.net`.

<img width="1516" height="453" alt="image" src="https://github.com/user-attachments/assets/6c4baece-f5e4-420a-a7d7-ff2801739175" />



---

## 3. Persistence

The attacker created a scheduled task named **WindowsUpdate**, pointing to `update.exe` for long-term access.

<img width="1640" height="347" alt="image" src="https://github.com/user-attachments/assets/4764d005-86dd-4d73-9f37-9fc3a10faaa8" />

<img width="1643" height="314" alt="image" src="https://github.com/user-attachments/assets/d8c1f08e-2832-4bcc-a8a4-4d2071742403" />

A backup domain account was created on the Domain Controller for fallback access.

<img width="1658" height="251" alt="image" src="https://github.com/user-attachments/assets/2939c749-8bc1-42ff-b672-211c2b20e966" />

AnyDesk was installed and configured for unattended remote access.

<img width="1319" height="244" alt="image" src="https://github.com/user-attachments/assets/c24be689-7325-4a79-9d8f-889af11a3f04" />


---

## 4. Privilege Escalation
The attacker:

- Bypassed UAC using `fodhelper.exe`  
- Escalated to SYSTEM privileges via Print Spooler  
- Dumped LSASS memory to steal Domain Admin credentials  
- Added `svc_backup` to the Admin group

<img width="1666" height="541" alt="image" src="https://github.com/user-attachments/assets/a8fadc55-9517-448a-b69b-7a9712f73a86" />

<img width="1645" height="211" alt="image" src="https://github.com/user-attachments/assets/4820a2b1-202d-46be-8dfb-3576a9bf0cbe" />

---
## 5. Defense Evasion

The attacker:

- Used legitimate Windows tools (LOLBins)  
- Cleared logs and temporary files  
- Executed malicious DLLs in memory  
- Maintained persistence with scheduled tasks and AnyDesk

<img width="1516" height="453" alt="image" src="https://github.com/user-attachments/assets/16784f8c-4f13-4940-97d0-26fea543a8fc" />

<img width="1641" height="452" alt="image" src="https://github.com/user-attachments/assets/44260147-9bd7-499c-ac58-09c50a40d5fb" />

<img width="1642" height="258" alt="image" src="https://github.com/user-attachments/assets/1e1345e0-da04-4da7-8298-3c9db361e93c" />

<img width="1630" height="208" alt="image" src="https://github.com/user-attachments/assets/6abe4f3a-e5ff-4ebb-82e8-af498bd1796b" />

---

## 6. Credential Access
The attacker:

- Used legitimate Windows tools (LOLBins)  
- Cleared logs and temporary files  
- Executed malicious DLLs in memory  
- Maintained persistence with scheduled tasks and AnyDesk

<img width="1688" height="346" alt="image" src="https://github.com/user-attachments/assets/973c3f0e-d673-4163-8b2b-ea13d8031492" />

<img width="1636" height="331" alt="image" src="https://github.com/user-attachments/assets/077fe933-5141-4646-84cd-4b7831581abe" />




---

## 7. Discovery
Attacker ran commands to map the network and find accounts:

<img width="1590" height="407" alt="image" src="https://github.com/user-attachments/assets/a56a2945-7c06-40b3-8304-b3937f9a1aa8" />

<img width="1371" height="403" alt="image" src="https://github.com/user-attachments/assets/77316d09-1cec-41e9-93a4-207e01537d67" />

<img width="1356" height="204" alt="image" src="https://github.com/user-attachments/assets/4f6b9273-a71f-4647-a94c-87332b45cf2f" />

---

## 8. Lateral Movement

Used stolen Domain Admin credentials to access other systems via SMB and admin shares.

<img width="1646" height="246" alt="image" src="https://github.com/user-attachments/assets/6290ffe5-5fd8-4cd1-a5b2-dd5371bb060d" />

<img width="985" height="244" alt="image" src="https://github.com/user-attachments/assets/c5be8b0e-157d-4fa2-a5d6-8cf20fe98d41" />



---

## 9. Data Collection

Sensitive files staged for exfiltration.

<img width="1609" height="204" alt="image" src="https://github.com/user-attachments/assets/a957e393-2223-4b2e-9f5f-ccec8dcfe300" />


---

## 10. Command & Control
Sliver DLL maintained a persistent connection to the C2 server.

<img width="1175" height="429" alt="image" src="https://github.com/user-attachments/assets/2e02eede-f957-486d-afd1-80e7f23eb0a7" />


---

## 11. Data Exfiltration
Files compressed and sent to MEGA.

<img width="1586" height="283" alt="image" src="https://github.com/user-attachments/assets/38f01f35-9bbb-4c4e-a0ad-76e1fd7af566" />



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
