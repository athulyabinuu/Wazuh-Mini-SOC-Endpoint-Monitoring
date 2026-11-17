
# Wazuh Mini-SOC – Endpoint Monitoring

## Project Overview

This repository documents **Internship Task 2 – Operation Mini-SOC: Endpoint Defense Challenge**, completed as part of the Cyber Security Analyst Internship Program at ICT Academy of Kerala (ICTAK).

The project demonstrates the deployment of a centralized Mini Security Operations Center (Mini-SOC) using Wazuh in an Oracle VirtualBox environment.

The lab focuses on endpoint monitoring, File Integrity Monitoring (FIM), security event analysis, SSH brute-force detection, Windows account monitoring, custom detection rules, and automated firewall response.

## Objectives

- Deploy Wazuh Manager, Indexer, and Dashboard.
- Register and monitor Windows and Ubuntu endpoints.
- Configure File Integrity Monitoring.
- Investigate Linux file activity using auditd.
- Detect repeated SSH authentication failures.
- Monitor Windows account creation and modification.
- Configure and test custom Wazuh detection rules.
- Implement automated firewall blocking and unblocking.
- Analyze security alerts using Wazuh dashboards.
- Explore MITRE ATT&CK mapping and vulnerability detection.

## Lab Architecture

The Mini-SOC environment was deployed using four virtual machines.

| Host | Operating System | IP Address | Role |
|---|---|---|---|
| Host A | Ubuntu 22.04 | 10.0.2.3 | Wazuh Manager, Indexer, Dashboard |
| Host B | Windows 11 | 10.0.2.15 | Windows Wazuh Agent |
| Host C | Ubuntu 22.04 | 10.0.2.4 | Ubuntu Wazuh Agent |
| Host D | Kali Linux | 10.0.2.5 | Security Testing Machine |

**Virtualization:** Oracle VirtualBox  
**Network Configuration:** NAT  
**Wazuh Version:** 4.14.10

## Tools and Technologies

- Wazuh Manager and Agent
- Wazuh Indexer and Dashboard
- Ubuntu Linux 22.04
- Windows 11
- Kali Linux
- Oracle VirtualBox
- Linux auditd
- Microsoft Defender
- MITRE ATT&CK
- Linux Firewall / Active Response

## Tasks Performed

### 1. Wazuh Deployment and Agent Registration

- Deployed Wazuh Manager, Indexer, and Dashboard.
- Installed Wazuh agents on Windows 11 and Ubuntu 22.04.
- Registered both endpoints with the central manager.
- Verified that both agents were active in the dashboard.

**Result:** Successfully established centralized endpoint monitoring.

### 2. File Integrity Monitoring (FIM)

Configured FIM to monitor important files and directories on both Windows and Ubuntu endpoints.

**Ubuntu Monitoring:**
- Monitored `/etc/passwd` and `/etc/shadow`.
- Enabled real-time monitoring for `/var/www/html`.
- Tested file creation, modification, and deletion using `honeypot.txt`.

Observed Wazuh rules:

| Rule ID | Event |
|---|---|
| 554 | File added |
| 550 | File integrity checksum changed |
| 553 | File deleted |

**Windows Monitoring:**
- Configured real-time monitoring for the Documents directory.
- Tested file activity using `honeypot2.txt`.
- Verified file-added activity using Wazuh rule 554.
- Verified modification and deletion events in manager logs.

### 3. Linux Audit Monitoring

Enabled auditd on the Ubuntu endpoint to monitor activity involving `/etc/passwd`.

Investigated a controlled file operation to identify process and user information.

| Field | Observed Value |
|---|---|
| Monitored File | /etc/passwd |
| Command | touch |
| Executable | /usr/bin/touch |
| Process ID (PID) | 7490 |
| User ID (UID) | 0 (root) |
| Audit Key | passwd_watch |

The test changed the file modification time without modifying account entries or permissions.

### 4. SSH Brute-Force Detection

Performed controlled SSH authentication failure testing from Kali Linux against the Ubuntu endpoint.

**Testing Environment:**
- Attacker: Kali Linux (10.0.2.5)
- Target: Ubuntu 22.04 (10.0.2.4)
- Target Username: athulya

Observed Wazuh detection rules:

| Rule ID | Description | Alert Level |
|---|---|---|
| 5760 | SSH authentication failed | 5 |
| 5766 | Repeated SSH authentication failures | 5 |
| 2502 | User missed the password more than one time | 10 |

Rule 2502 generated a Level 10 alert associated with:

- MITRE ATT&CK Tactic: Credential Access
- Technique: T1110 – Brute Force
- Sub-technique: T1110.001 – Password Guessing

**Result:** Repeated SSH authentication failures were successfully detected.

### 5. Windows Account Monitoring

Monitored Windows account activities on the Windows 11 endpoint.

Created and investigated the test account `soc-labadmin`.

Observed Windows Security Event IDs:

| Event ID | Description |
|---|---|
| 4720 | User account created |
| 4722 | User account enabled |
| 4738 | User account changed |

Relevant Wazuh detection rules:

- Rule 60109 – Windows account creation activity.
- Rule 60118 – Windows account modification.

The test account was also verified as a member of the Windows Administrators group.

**Result:** Windows account creation and modification activities were successfully detected.

### 6. Base64 Payload Analysis

Decoded a Base64-encoded PowerShell command supplied in the assignment.

The decoded command attempted to download a PowerShell script from a remote URL and execute it in memory.

The command was inspected for security analysis and was not executed.

**Result:** Successfully performed static analysis of the decoded command.

### 7. Wazuh Active Response

Configured Wazuh Active Response using the `firewall-drop` command to respond to SSH brute-force alerts.

**Configuration:**

| Setting | Value |
|---|---|
| Command | firewall-drop |
| Target Agent | 003 – Host-C-Ubuntu |
| Trigger Rule | 2502 |
| Timeout | 60 seconds |
| Response Location | defined-agent |

Observed response rules:

| Rule ID | Description |
|---|---|
| 651 | Host blocked by firewall-drop |
| 652 | Host unblocked by firewall-drop |

The Kali source IP (10.0.2.5) was temporarily blocked following the SSH brute-force detection.

The block was automatically removed after approximately 60 seconds.

**Result:** Active Response blocking and unblocking successfully verified.

### 8. Custom Wazuh Rule Configuration

Created custom detection rules in:

`/var/ossec/etc/rules/local_rules.xml`

Custom rules included:

| Rule ID | Alert Level | Description |
|---|---|---|
| 100001 | 5 | SSH authentication failure from a specified IP |
| 100002 | 10 | Repeated SSH authentication failures detected |

Tested the custom SSH detection logic using:

`/var/ossec/bin/wazuh-logtest`

The simulated event successfully matched custom rule 100002.

**Result:** Custom rule matching verified using the Wazuh log testing utility. This was a simulated rule test, not a separate live attack alert.

### 9. SOC Dashboard Analysis

Reviewed the following Wazuh dashboard capabilities:

**File Integrity Monitoring**
- Reviewed file activity across Windows and Ubuntu endpoints.
- Investigated file creation and modification events.

**MITRE ATT&CK**
- Reviewed the mapping of detected SSH brute-force activity.
- Explored attacker tactics and techniques.

**Vulnerability Detection**
- Reviewed vulnerability findings reported for monitored endpoints.
- Examined severity categories and dashboard information.

The dashboard findings were recorded as reported at the time of inspection, not as counts of unique confirmed exploitable vulnerabilities.

## EICAR Endpoint Security Validation

Reviewed Microsoft Defender security configuration on the Windows 11 endpoint.

The following settings were observed:

- Real-time protection enabled.
- Antivirus protection enabled.
- Security intelligence definitions up to date.

The EICAR activity was not fully validated as a confirmed detection or quarantine event. Therefore, no successful EICAR detection is claimed in this repository.

## Key Results

| Activity | Result |
|---|---|
| Wazuh Server Deployment | Successful |
| Windows Agent Registration | Verified |
| Ubuntu Agent Registration | Verified |
| Ubuntu FIM | Successful |
| Windows FIM | Verified |
| Linux Audit Monitoring | Verified |
| SSH Brute-Force Detection | Successful |
| Windows Account Monitoring | Successful |
| Custom Rule 100002 | Successfully Tested |
| Active Response Blocking | Successful |
| Automatic IP Unblocking | Successful |
| SOC Dashboard Analysis | Completed |
| EICAR Detection | Not Fully Validated |

## Key Learning Outcomes

Through this internship task, I gained practical experience in:

- SIEM and SOC fundamentals.
- Centralized endpoint monitoring.
- Wazuh deployment and agent management.
- File Integrity Monitoring.
- Linux audit log investigation.
- SSH brute-force detection.
- Windows security event analysis.
- Custom detection rule configuration.
- Automated incident response.
- Firewall-based IP blocking.
- MITRE ATT&CK mapping.
- Vulnerability monitoring and dashboard analysis.

## Challenges and Limitations

- EICAR detection and quarantine were not fully validated.
- Separate Windows FIM modification and deletion screenshots were unavailable, although the events were verified in manager logs.
- Custom Windows account rule 100003 was not included among the verified custom-rule results.
- The manager's separate active-responses.log file was empty during inspection; response evidence was verified through alerts.json and the Host C response log.
- The custom SSH rule test was performed using wazuh-logtest and was not presented as a separate live attack alert.

## Project Documentation

The complete technical implementation report includes:

- Mini-SOC architecture and deployment.
- Endpoint registration and monitoring.
- FIM and Linux audit investigation.
- SSH brute-force detection.
- Windows account monitoring.
- Active Response configuration and verification.
- Custom Wazuh rules.
- SOC dashboard analysis.
- Screenshot evidence and findings.

**Technical Report:** [Operation Mini-SOC – Internship Task 2](Operation_Mini_SOC___Internship_Task_2___Athulya_Binu.pdf)

## Internship Details

**Organization:** ICT Academy of Kerala (ICTAK)

**Program:** Cyber Security Analyst Internship

**Track:** Blue Team

**Assignment:** Internship Task 2 – Operation Mini-SOC

**Prepared By:** Athulya Binu

**Qualification:** B.Sc Computer Science

**Lab Platform:** Oracle VirtualBox

**Submission Date:** 12 November 2026

## Disclaimer

This project was conducted in a controlled virtual lab environment for educational and cybersecurity learning purposes.

All attack simulations and security testing activities were performed within the isolated lab environment.

---

**Author:** Athulya Binu

**GitHub:** https://github.com/athulyabinuu

**LinkedIn:** https://www.linkedin.com/in/athulyabinu/
