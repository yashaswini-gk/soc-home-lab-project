# **SOC Home Lab: Sysmon + Splunk Detection Pipeline** 

A hands-on Security Operations Center (SOC) lab built from scratch to simulate attacker activity and detect it using an enterprise-standard SIEM workflow. 


## Overview 
This project simulates a mini enterprise security monitoring environment. I built an isolated virtual network with a "victim" Windows machine and an "attacker" Kali Linux machine, instrumented the victim with detailed endpoint logging, ingested those logs into a SIEM, and demonstrated end-to-end detection of simulated attacker behavior from network reconnaissance to post-exploitation commands.

## Architecture
┌─────────────────┐         Internal Network         
┌──────────────────────┐ 
 
│   Kali Linux     │ ───────────────────────────────► │     Windows 11        │ 
 
│  (Attacker VM)    │      192.168.100.20/24            │    (Victim VM)          │ 
 
│                    │                                    │  192.168.100.10/24     │ 
 
│  - Nmap            │                                    │  - Sysmon (logging)     │ 
 
│  - Recon tools      │                                   │  - Splunk (SIEM)         │ 
 
└─────────────────┘                                    └──────────┬───────────┘ 
 
                                                                   │ 
 
                                                                   ▼ 
 
                                                         ┌───────────────────────┐ 
 
                                                        │  Splunk Dashboard        │ 
 
                                                        │  (Search & Detection)    │
                                                        
                                                          └───────────────────────┘

## Tools Used
| Tool | Purpose |
|------|---------|
| VirtualBox | Virtualization platform for isolated lab environment |
| Windows 11 | Target/victim machine |
| Kali Linux | Attacker machine for simulating recon and post-exploitation activity |
| Sysmon (SwiftOnSecurity config) | Endpoint telemetry — process creation, network connections |
| Splunk Enterprise | SIEM — log ingestion, indexing, and search |
| Nmap | Network reconnaissance/port scanning |

## What I Built
1. Isolated lab network — Windows and Kali VMs connected via a VirtualBox Internal 
Network, fully isolated from the host machine's internet connection, with static IPs assigned to each. 
2. Endpoint instrumentation — Installed Sysmon on the Windows VM using a 
community-maintained configuration (SwiftOnSecurity) to capture high-fidelity security 
event data (process creation, network connections) beyond what Windows logs by 
default. 
3. SIEM ingestion pipeline — Configured Splunk Enterprise to continuously monitor and 
index the Sysmon Operational log via a custom inputs.conf configuration. 
4. Attack simulation — From the Kali VM, performed:
   - Network reconnaissance (nmap -sV) against the Windows target
   - Simulated post-exploitation commands directly on the target (whoami, net user) — commands commonly run by an attacker immediately after gaining initial access, to enumerate the current user and local accounts 
6. Detection — Used Splunk's search language (SPL) to locate and verify the simulated 
attacker activity in the ingested logs.

## Detection Examples
Query used to detect reconnaissance commands (MITRE T1033):
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 
CommandLine="*whoami*"

Query used to detect account enumeration (MITRE T1087.001): 
source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 
CommandLine="*net user*"

Both queries successfully surfaced the corresponding Sysmon Event ID 1 (Process Creation) events, including the full command line, parent process, and executing user.

## (Screenshots below)
- - - - 
screenshots/splunk-dashboard.png — Splunk ingesting live Sysmon data 
screenshots/nmap-scan.png — Nmap scan from Kali against the Windows target 
screenshots/whoami-detection.png — Detected whoami execution in Splunk 
screenshots/netuser-detection.png — Detected net user execution in Splunk

## MITRE ATT&CK Mapping

| Simulated Activity | Technique ID | Technique Name |
|---|---|---|
| 'whoami' | T1033 | System Owner/User Discovery |
| 'net user' | T1087.001 | Account Discovery: Local Account |
| Nmap scan | T1046 | Network Service Discovery |

## Scale & Coverage
Log volume: ~18,000 Sysmon events indexed during testing
Log sources: 2 (Process Creation – EventCode 1, Network Connection – EventCode 3)
Detection queries validated: 2 (discovery-phase command execution)
MITRE ATT&CK techniques covered: T1033, T1087.001, T1046


## Skills Demonstrated - 
- SIEM configuration and log source onboarding (Splunk inputs.conf, Windows Event Log channels) 
- Endpoint detection engineering (Sysmon configuration and deployment) 
- Search Processing Language (SPL) for threat hunting and log analysis 
- Virtual network isolation and static IP configuration for lab safety 
- Basic offensive reconnaissance techniques (Nmap) to inform defensive detection logic 
- Troubleshooting real-world SIEM ingestion issues (log channel registration, service 
restarts, config file placement)

## Lessons Learned
- Windows Firewall silently drops most unsolicited connection attempts, meaning many 
network-based attacks won't generate network connection logs unless a service actually 
accepts the connection — a useful reminder that detection coverage depends heavily on 
what's actually listening and logging. 
- Splunk's Windows Event Log picker only detects log channels present at service startup; new channels (like Sysmon's) require a service restart to appear, or manual 
configuration via inputs.conf. 
- Process-based detection (Sysmon Event ID 1) proved more reliable than network-based 
detection in this isolated lab setup, reflecting how real SOC teams often rely on multiple overlapping data sources rather than a single detection method.

## Future Improvements
- Build custom Splunk detection rules/alerts that trigger automatically on suspicious 
command patterns 
- Add a second attack scenario (e.g., simulated brute-force login attempts) 
- Expand to a full Wazuh deployment for comparison against Splunk 
- Add MITRE ATT&CK technique mapping to each detection (e.g., whoami → T1033 System Owner/User Discovery)


Built as a personal cybersecurity learning project to develop hands-on SOC Analyst and Digital Forensics skills
