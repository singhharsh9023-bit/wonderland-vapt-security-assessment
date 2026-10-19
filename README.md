# Wonderland VAPT — Full Security Assessment

A documented Vulnerability Assessment and Penetration Testing (VAPT) exercise against the TryHackMe Wonderland lab, covering network reconnaissance, web enumeration, steganography, credential exposure, SSH access, Linux privilege escalation, evidence collection, Cyber Kill Chain mapping, and defensive remediation.

---

## Project Information

| Field | Details |
|---|---|
| Project Title | Modeling Directory Traversal and Linux Executable Capability Exploitation in Corporate Endpoints |
| Assessment | Wonderland VAPT |
| Organization Scenario | Axion Data Corp |
| Prepared By | Harsh Raj Singh |
| Student Platform | TryHackMe |
| Attacking Machine | Kali Linux |
| Kali Username | `harsh` |
| Kali Hostname | `axion-data-corp` |
| Target Machine Hostname | `wonderland` |
| Report | `Wonderland_VAPT_Report.pdf` |

## Project Overview

This project evaluates the security of the TryHackMe Wonderland lab in a controlled environment. The assessment investigates exposed network services, discoverable web resources, concealed information in image files, credentials exposed in HTML source code, and Linux privilege-escalation weaknesses.

The documented attack path progresses from reconnaissance and web content discovery to SSH access as `alice`, followed by the reported privilege transitions `alice → rabbit → hatter → root`.

The findings are mapped to the Cyber Kill Chain framework, and defensive recommendations are provided to help prevent similar security weaknesses.

## Lab Environment and IP Addresses

| Component | Value |
|---|---|
| Target platform | TryHackMe |
| Room | Wonderland |
| Room URL | https://tryhackme.com/room/wonderland |
| TryHackMe profile | https://tryhackme.com/p/singhharsh9023 |
| Attacker VPN interface | `tun0` |
| Attacker VPN IP | `192.168.166.209` |
| Target IP in the main report | `10.48.158.123` |
| IP in the historical raw Nmap log | `10.48.146.123` |

**IP address verification note:** The report lists `10.48.158.123` in the target setup, web enumeration, and SSH sections. Its raw Nmap log on page 25 records `10.48.146.123`. This discrepancy should be checked against the original scan output. TryHackMe target IPs may change when a machine is restarted, so use the currently assigned IP for any new authorized testing.

The attacking machine was configured with the hostname `axion-data-corp`, and the username was `harsh`. OpenVPN was used to connect Kali Linux to the TryHackMe lab network.

## Tools and Technologies

- **Kali Linux** — Penetration-testing environment
- **OpenVPN** — Secure connection to the TryHackMe lab
- **Nmap** — Network scanning and service detection
- **Gobuster** — Web directory enumeration
- **ExifTool** — Image metadata analysis
- **Steghide** — Steganography investigation
- **SSH** — Authenticated remote access
- **Linux command-line tools** — User, filesystem, permission, SUID, and capability enumeration
- **Python** — Investigation of module search-path hijacking
- **Perl** — Investigation of Linux `cap_setuid` capability abuse
- **Google Sheets** — Proof-of-concept evidence archive
- **GitHub** — Project report and evidence repository

## Assessment Methodology

### 1. Environment Setup and VPN Connectivity

- Configured Kali Linux for the assessment.
- Set the Kali hostname to `axion-data-corp`.
- Connected to the TryHackMe network using OpenVPN.
- Verified the `tun0` interface and attacker VPN IP.

### 2. Network Reconnaissance — Nmap

Performed service discovery and version detection using Nmap.

Documented command:

`nmap -sV -sC -oN wonderland_scan.txt <TARGET_IP>`

Reported services:

| Port | Service | Reported version/details |
|---|---|---|
| 22/tcp | SSH | OpenSSH 7.6p1 Ubuntu |
| 80/tcp | HTTP | Go-based HTTP server |

The web application displayed the title **“Follow the white rabbit.”**

### 3. Web Enumeration — Gobuster

Enumerated web directories and identified:

- `/img`
- `/index.html`
- `/r`

The `/img` directory exposed image files that were investigated during the subsequent steganography phase.

### 4. Image Discovery and Steganography

Downloaded and examined the following images:

- `alice_door.jpg`
- `alice_door.png`
- `white_rabbit_1.jpg`

ExifTool was used to inspect metadata, while Steghide was used to investigate concealed data in `white_rabbit_1.jpg`.

The report records extraction of `hint.txt`, containing a clue that led to the hidden path:

`/r/a/b/b/i/t`

### 5. Hidden Web Path and Credential Exposure

Inspected the discovered page and its HTML source code. The report documents credentials stored inside an HTML element hidden with CSS.

This finding illustrates the risk of placing sensitive authentication information in client-accessible content.

Sensitive credential values are intentionally omitted from this public README.

### 6. Initial Access — SSH

Used the credentials discovered during the assessment to establish SSH access as `alice`.

Reported target operating system:

- Ubuntu 18.04.4 LTS
- Linux kernel `4.15.0-101-generic`

The report documents successful initial access and subsequent host enumeration.

### 7. Linux Host Enumeration

Investigated:

- Local user accounts and home directories
- File ownership and permissions
- Sudo permissions
- SUID-enabled binaries
- Linux capabilities
- Cron jobs
- Restricted flag-file access

The report identifies the users `alice`, `rabbit`, `hatter`, and `tryhackme`.

### 8. Privilege Escalation

The report documents the following privilege-escalation chain.

#### Stage 1 — Alice to Rabbit

A sudo rule allowed a Python script to run as `rabbit`. The report describes Python module search-path hijacking involving the script's `random` module.

#### Stage 2 — Rabbit to Hatter

The SUID-enabled `teaParty` executable was found to invoke `date` without an absolute path. The report documents a PATH-hijacking technique associated with this executable.

#### Stage 3 — Hatter to Root

The report identifies the Perl interpreter's `cap_setuid` capability and documents its use in obtaining root-level access.

#### Root Verification

The report records root verification using `whoami` and `id`, along with recovery of the lab's user and root flags.

The detailed terminal screenshots and supporting evidence should be reviewed alongside the PDF.

## Key Findings

| Finding | Security Impact |
|---|---|
| Exposed SSH and HTTP services | Increases the externally discoverable attack surface |
| Hidden web resources | Can expose additional application content |
| Steganographic content in an image | Concealed information can reveal further attack clues |
| Credentials in client-accessible HTML | May enable unauthorized authenticated access |
| Unsafe sudo delegation and Python module loading | Can allow privilege escalation |
| SUID executable using an untrusted PATH resolution | Can create a privileged command-execution risk |
| Unnecessary `cap_setuid` capability on Perl | Can enable escalation to root |
| Legacy operating-system version | Requires support-status and patch review |

## Cyber Kill Chain Mapping

The report maps observed activities to the Cyber Kill Chain while distinguishing confirmed activities from stages not independently established.

| Stage | Assessment mapping |
|---|---|
| Reconnaissance | Nmap scanning, Gobuster enumeration, and web content discovery |
| Weaponization | A distinct preparation stage was not independently established |
| Delivery | Image retrieval and hidden-page discovery were documented; separate payload delivery was not established |
| Exploitation | Exposed credentials were used for SSH access; privilege escalation was investigated |
| Installation | No persistence mechanism was established |
| Command and Control | SSH access was documented; separate C2 infrastructure was not established |
| Actions on Objectives | Root access and lab flags were reported; supporting evidence should be retained |

## Defensive Recommendations

1. Remove credentials and sensitive information from client-accessible web content.
2. Rotate credentials that may have been exposed.
3. Enforce least-privilege sudo rules and restrict execution of privileged scripts.
4. Secure Python module loading and dependency search paths.
5. Ensure privileged executables use trusted absolute paths for external commands.
6. Audit SUID binaries and remove unnecessary SUID permissions.
7. Remove unnecessary Linux capabilities from interpreters and other general-purpose executables.
8. Monitor authentication events, privilege transitions, and modifications to privileged files.
9. Review operating-system support status and plan upgrades where necessary.
10. Retest the environment after remediation and retain evidence of the results.

## Project Deliverables

### 1. Penetration Testing Report

The PDF report contains the assessment methodology, environment configuration, findings, command records, privilege-escalation narrative, Cyber Kill Chain mapping, and remediation recommendations.

**Report filename:** `Wonderland_VAPT_Report.pdf`

### 2. Proof-of-Concept Evidence

The `Wonderland_POC_Evidence/` directory contains supporting screenshots for documented assessment stages, including reconnaissance, web enumeration, image analysis, credential discovery, SSH access, Linux enumeration, and privilege escalation.

### 3. Google Sheets PoC Archive

The report references a Google Sheets archive containing phase-by-phase commands, screenshot links, and notes.

[Open the Wonderland PoC Evidence Archive](https://docs.google.com/spreadsheets/d/13J_wD4ImBfR0ryaJXgQTrAZfMRczM3UX_yX8UerqFLA/edit?usp=sharing)

The archive link is transcribed from the uploaded report. Verify that it opens the correct sheet and that view-only access is configured before submission.

### 4. TryHackMe Room and Profile

- [Wonderland Room](https://tryhackme.com/room/wonderland)
- [TryHackMe Profile — Harsh Raj Singh](https://tryhackme.com/p/singhharsh9023)

## Repository Structure

```text
wonderland-vapt-security-assessment/
├── README.md
├── Wonderland_VAPT_Report.pdf
├── Wonderland_POC_Evidence/
│   ├── Step_01_Nmap_Scan.png
│   ├── Step_02_Gobuster_Directory_Scan.png
│   ├── Step_03_Website_Images_Download.png
│   ├── Step_04_Exiftool_Metadata.png
│   ├── Step_05_Steghide_Extraction.png
│   └── ...additional evidence screenshots
└── Wonderland_POC_Archive/
    └── G19_Wonderland_POC_Evidence.xlsx
```

This is the recommended structure. Keep only filenames and folders that actually exist in the GitHub repository.

## Learning Outcomes

- Network and web reconnaissance
- Web directory enumeration
- Image metadata and steganography analysis
- Credential-exposure identification
- SSH-based initial access
- Linux user and permission enumeration
- Sudo, SUID, PATH, and Linux capability security
- Privilege-escalation investigation
- Cyber Kill Chain mapping
- Evidence organization and security remediation reporting

## Scope and Ethical Disclaimer

This project documents activity performed against the designated TryHackMe Wonderland lab for educational purposes. It is not evidence of a real-world compromise of Axion Data Corp or another production organization.

All testing should be restricted to systems for which explicit authorization has been granted. Sensitive credentials, VPN configuration files, session data, and other secrets should not be committed to a public repository.

## References

- [TryHackMe — Wonderland](https://tryhackme.com/room/wonderland)
- [MITRE ATT&CK — Enterprise Linux Matrix](https://attack.mitre.org/matrices/enterprise/linux/)
- [MITRE ATT&CK — Abuse Elevation Control Mechanism](https://attack.mitre.org/techniques/T1548/)
- [Lockheed Martin — Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)

---

**Author:** Harsh Raj Singh  
**Project:** Wonderland VAPT Security Assessment
