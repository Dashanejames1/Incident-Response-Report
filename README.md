# Incident-Response-Report



**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2]  
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
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable 2] — `192.168.79.130` |
| SIEM |	Splunk Enterprise (Free) |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |


> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.


---

## 🔬 Tasks / Assessments Performed

### 1. [Incident Summary]

On September 23-24, 2026, a simulated SSH brute force attack was conducted against Metasploitable 2 (192.168.79.130) from a Kali Linux attacker machine (192.168.79.129). The attack used Medusa with a targeted wordlist and successfully cracked the SSH credentials (msfadmin:msfadmin) in under one second. The incident was detected by a pre-configured Splunk SSH Brute Force Detection alert that fired on ingested Medusa log data. Network traffic was simultaneously captured via TCPDump and imported into Splunk, where SPL queries confirmed the attack pattern and identified the compromised account.

### 2. [Timeline]

Time |Attacker |Action	|ATT&CK Technique
22:05	| TCPDump | capture started on eth0	|T1040 - Network Sniffing
22:06	|Nmap |port scan against 192.168.79.130	|T1046 - Network Service Discovery
22:29	|Hydra |brute force attempted — failed (kex error) |T1110 – Brute Force
22:34	|Medusa |brute force started with rockyou.txt	|T1110.001 – Password Guessing
22:45	|Switched to targeted wordlist	|T1110.001 – Password Guessing
12:54:28	|Medusa |began authentication attempts	|T1110.001 – Password Guessing
12:54:29	|msfadmin:msfadmin cracked — SSH access gained|	T1078 – Valid Accounts
12:54:32	|Session complete — root-level access confirmed|	T1059 – Command and Scripting Interpreter


### 3. [Impact Assesment]

 The compromised system (Metasploitable 192.168.79.130) runs multiple critical services including FTP (21), SSH (22), HTTP (80), MySQL (3306), and several RPC services. Full root-level access was obtained meaning an attacker could read, modify or delete all data on the system, create backdoor accounts, pivot to other network hosts, install malware or ransomware, and exfiltrate sensitive data. In a real environment this level of access would constitute a critical severity incident requiring immediate escalation.



### 4. [Containment and eradification steps taken.]

Containment — attacker IP blocked at the firewall:

sudo iptables -A INPUT -s 192.168.79.129 -j DROP
sudo iptables -A OUTPUT -d 192.168.79.129 -j DROP

All active SSH sessions from the attacker IP were terminated immediately. SSH access was restricted to trusted IPs only pending investigation.

Eradication — the following items were removed or remediated: default msfadmin credentials changed to a strong unique password, SSH authorized_keys file audited for unauthorized entries, all user accounts and scheduled tasks reviewed for unauthorized changes, and a mandatory credential change policy established for all newly deployed systems.



### 5. [Root Cause]

The attack succeeded because of two compounding failures: first, default credentials (msfadmin:msfadmin) were never changed after deployment — the same username and password made the account trivially easy to crack. Second, no SSH account lockout policy was configured, allowing unlimited login attempts without rate limiting or temporary suspension. Either control alone would have significantly slowed or prevented the attack entirely. Together their absence allowed a complete compromise in under one second.



### 6. [3 Specific security controls to prevent recurrence.]

1. Enforce a default credential elimination policy — require credential changes on all systems before network connectivity is established. Automated scanning tools should verify no default credentials exist on any deployed system as part of the onboarding checklist. This single control would have prevented this incident entirely.

2. Implement SSH account lockout — configure SSH to lock accounts after 5 failed attempts within 60 seconds using fail2ban or PAM configuration. This stops brute force attacks before they succeed even when a correct password exists in the attacker’s wordlist.

3. Deploy real-time SIEM alerting — upgrade from Splunk Free (hourly scheduling) to a production SIEM with real-time alert capabilities. This attack succeeded in under one second — an hourly alert window means a breach could go undetected for up to 59 minutes. Real-time detection is not optional in a production SOC environment.


**Findings:**


🗺️ MITRE ATT&CK Mapping

Action Performed	ATT&CK Tactic	Technique ID	Technique Name

TCPDump capture of lab traffic	Collection	T1040	Network Sniffing

Nmap port scan of the target	Discovery	T1046	Network Service Discovery

Hydra / Medusa password guessing over SSH	Credential Access	T1110.001	Brute Force: Password Guessing

msfadmin:msfadmin accepted — SSH access gained	Initial Access	T1078	Valid Accounts

Root shell on the target	Execution

---


🔑 Technical Notes

Hydra vs. Medusa: Hydra failed with a key-exchange (kex) error against Metasploitable's older SSH; Medusa completed the brute force successfully, so it became the tool of record for this incident.

Targeted wordlist beat rockyou.txt: Medusa cracked the account almost instantly once switched to a short targeted wordlist, since the password matched the username.

Splunk Free scheduling limit: Splunk Free only supports scheduled (not real-time) alerts, so the detection ran on an hourly window. This is the basis for recommendation #3.

---

## 📌 About This Project

This is the detection-and-response capstone of my cybersecurity portfolio. Earlier repositories covered scanning, capturing, and exploiting from the attacker's side; this one takes the analyst's view — detecting an attack in a SIEM and working it through the full incident response lifecycle, which is the work I'm aiming for as a cybersecurity analyst.


## Related repositories:

Nmap-Host-Discovery-and-Lab-Baseline — Established a baseline of the lab network using ping sweeps and host discovery
Service-Version-Detection-and-OS-Fingerprinting — Identified exact software versions on Metasploitable and researched the CVEs tied to them
Wireshark-Capture-and-Analyze-Traffic — Captured and analyzed live packet traffic, including plaintext credential exposure over Telnet
TCPDump-CLI-Packet-Capture — Captured traffic from the command line and saved it to a .pcap file for analysis in Wireshark
Install-Snort-IDS-and-Write-Detection-Rules — Installed Snort 3 and wrote a custom rule to alert on ICMP traffic between the lab machines


##👤 Author

Dashane James
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology — SUNY Canton
🏆 CompTIA Security+ | CySA+ (In Progress)
🔗 GitHub | Zero Trust Cyber Security Brand

This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.
