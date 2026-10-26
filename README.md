# Incident-Response-Report


**Author:** Dashane James
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2
**Purpose:** Detect, investigate, and document a simulated SSH brute force attack using Splunk, then document the full incident response lifecycle.

**Status:** 🔵 Completed

---

## 📋 Overview

This repository documents a full incident response writeup for a simulated SSH brute force attack in an isolated lab. Using Medusa from Kali, the default msfadmin:msfadmin credentials on Metasploitable 2 were cracked in under a second. A pre-configured Splunk "SSH Brute Force Detection" alert fired on the ingested log data, and TCPDump traffic imported into Splunk was queried with SPL to confirm the attack pattern and identify the compromised account. The report follows the incident through summary, timeline, impact, containment and eradication, root cause, and recommended controls.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1, `192.168.79.129` |
| Target Machine | Metasploitable 2, `192.168.79.130` |
| SIEM | Splunk Enterprise (Free) |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11, ASUS Vivobook 14 |


> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.


---

## 🔬 Tasks / Assessments Performed

### 1. Incident Summary

On September 23 and 24, 2026, a simulated SSH brute force attack was conducted against Metasploitable 2 (192.168.79.130) from the Kali Linux attacker machine (192.168.79.129). The attack used Medusa with a targeted wordlist and successfully cracked the SSH credentials (msfadmin:msfadmin) in under one second. The incident was detected by a pre-configured Splunk SSH Brute Force Detection alert that fired on ingested Medusa log data. Network traffic was simultaneously captured via TCPDump and imported into Splunk, where SPL queries confirmed the attack pattern and identified the compromised account.

### 2. Timeline

| Time | Actor | Action | ATT&CK Technique |
|---|---|---|---|
| 22:05 | TCPDump | Capture started on eth0 | T1040 - Network Sniffing |
| 22:06 | Nmap | Port scan against 192.168.79.130 | T1046 - Network Service Discovery |
| 22:29 | Hydra | Brute force attempted, failed (kex error) | T1110 - Brute Force |
| 22:34 | Medusa | Brute force started with rockyou.txt | T1110.001 - Password Guessing |
| 22:45 | Medusa | Switched to targeted wordlist | T1110.001 - Password Guessing |
| 12:54:28 | Medusa | Authentication attempts began | T1110.001 - Password Guessing |
| 12:54:29 | Medusa | msfadmin:msfadmin cracked, SSH access gained | T1078 - Valid Accounts |
| 12:54:32 | Attacker | Root shell confirmed on target | T1059.004 - Command and Scripting Interpreter: Unix Shell |


### 3. Impact Assessment

The compromised system (Metasploitable 192.168.79.130) runs multiple critical services including FTP (21), SSH (22), HTTP (80), MySQL (3306), and several RPC services. Full root-level access was obtained, meaning an attacker could read, modify, or delete all data on the system, create backdoor accounts, pivot to other network hosts, install malware or ransomware, and exfiltrate sensitive data. In a real environment this level of access would constitute a critical severity incident requiring immediate escalation.



### 4. Containment and Eradication Steps

**Containment:** attacker IP blocked at the firewall:

```bash
sudo iptables -A INPUT -s 192.168.79.129 -j DROP
sudo iptables -A OUTPUT -d 192.168.79.129 -j DROP
```

All active SSH sessions from the attacker IP were terminated immediately. SSH access was restricted to trusted IPs only pending investigation.

**Eradication:** the following items were removed or remediated: default msfadmin credentials changed to a strong unique password, SSH authorized_keys file audited for unauthorized entries, all user accounts and scheduled tasks reviewed for unauthorized changes, and a mandatory credential change policy established for all newly deployed systems.



### 5. Root Cause

The attack succeeded because of two compounding failures: first, default credentials (msfadmin:msfadmin) were never changed after deployment. The same username and password made the account trivially easy to crack. Second, no SSH account lockout policy was configured, allowing unlimited login attempts without rate limiting or temporary suspension. Either control alone would have significantly slowed or prevented the attack entirely. Together their absence allowed a complete compromise in under one second.



### 6. Three Specific Security Controls to Prevent Recurrence

1. **Enforce a default credential elimination policy:** Require credential changes on all systems before network connectivity is established. Automated scanning tools should verify no default credentials exist on any deployed system as part of the onboarding checklist. This single control would have prevented this incident entirely.

2. **Implement SSH account lockout:** Configure SSH to lock accounts after 5 failed attempts within 60 seconds using fail2ban or PAM configuration. This stops brute force attacks before they succeed even when the correct password exists in the attacker's wordlist.

3. **Deploy real-time SIEM alerting:** Upgrade from Splunk Free (hourly scheduling) to a production SIEM with real-time alert capabilities. This attack succeeded in under one second. An hourly alert window means a breach could go undetected for up to 59 minutes. Real-time detection is not optional in a production SOC environment.

---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| TCPDump capture of lab traffic | Collection | T1040 | Network Sniffing |
| Nmap port scan of the target | Discovery | T1046 | Network Service Discovery |
| Hydra / Medusa password guessing over SSH | Credential Access | T1110.001 | Brute Force: Password Guessing |
| msfadmin:msfadmin accepted, SSH access gained | Initial Access | T1078 | Valid Accounts |
| Root shell on the target | Execution | T1059.004 | Command and Scripting Interpreter: Unix Shell |

---

## 🔑 Technical Notes

**Hydra vs. Medusa:** Hydra failed with a key exchange (kex) error against Metasploitable's older SSH server because Hydra's modern libssh library dropped support for the legacy cryptographic algorithms Metasploitable requires. Medusa retains legacy algorithm support and completed the brute force successfully, so it became the tool of record for this incident.

**Targeted wordlist beat rockyou.txt:** Medusa cracked the account almost instantly once switched to a short targeted wordlist, since the password matched the username. The 14 million password rockyou.txt would have found it eventually, but a targeted list of default credentials is a more realistic first pass in real-world credential attacks.

**Splunk Free scheduling limit:** Splunk Free only supports scheduled (not real-time) alerts, so detection ran on an hourly window. This is the direct basis for recommendation 3 above. In a production environment, the same detection logic running every 60 seconds or in real time would have flagged the attack before the session was even established.

---

## 📌 About This Project

This report is the written deliverable that closes the SIEM, log analysis, and incident response phase. The companion repository, Incident-Response-Simulation, walks through the seven NIST SP 800-61 phases step by step. This document takes a different angle: it formats the same event as a formal incident report, the kind of artifact a SOC analyst produces for management, legal, and compliance audiences after an incident is resolved.

Being able to translate a technical investigation into a clear, structured report is one of the most important skills in security operations. The timeline, impact assessment, containment steps, root cause analysis, and control recommendations here follow the structure used in real incident reports, documenting both what happened and what needs to change to prevent it from happening again.

**Related repositories (SIEM, log analysis, and incident response phase):**
- `Splunk-Free-Installation-and-Ingest-First-Logs`: Installed the Splunk SIEM that served as the detection platform for this incident
- `Splunk-Search-Language`: SPL queries used to analyze the attack data and confirm the compromise
- `Generating-Attacks-and-Defending-in-Splunk`: Executed the SSH brute force attack documented in this report and built the detection alerts that caught it
- `Install-and-Understand-ELK-Stack`: Installed the ELK Stack as a second SIEM platform alongside Splunk
- `Install-Snort-IDS-and-Write-Detection-Rules`: Network-layer IDS that would catch the reconnaissance phase preceding this attack
- `Incident-Response-Simulation`: Step-by-step walkthrough of the NIST SP 800-61 response lifecycle for this same event

---

## 👤 Author

---

**Dashane James**
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology, SUNY Canton
🏆 CompTIA Security+
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
