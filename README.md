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

# ⚙️ Attack Flow (Simple)

## 1. Initial Access
User downloaded a malicious ISO file from a phishing email.

![Initial Access](images/initial-access.png)

---

## 2. Execution
Malware executed using `rundll32.exe`.

![Execution](images/execution.png)

---

## 3. Persistence
- Scheduled task created (`WindowsUpdate`)
- AnyDesk installed

![Persistence](images/persistence.png)

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
