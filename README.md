# Wazuh Challenge

**SIEM Deployment | Endpoint Monitoring | Threat Detection | Incident Investigation | SOAR Automation**

## Project Overview

The [**Wazuh Challenge**](https://www.youtube.com/watch?v=duEibRGYMHo&list=PLHmErnFy_2zk) from MyDFIR is an eight-module cybersecurity home lab focused on building and operating a Security Operations Center (SOC) monitoring environment. I deployed Wazuh, onboarded Windows and Ubuntu endpoints, generated and investigated telemetry, built security dashboards and custom detections, simulated post-compromise activity, and integrated automated alert enrichment and response.

## Tools and Technologies

| Category | Technologies |
|---|---|
| SIEM and log analysis | Wazuh, OpenSearch, Filebeat |
| Endpoints and telemetry | Windows 11, Ubuntu Linux, Wazuh agents, Sysmon |
| Monitoring and detection | Windows Security Events, Linux SSH logs, File Integrity Monitoring (FIM), Wazuh custom rules |
| Adversary simulation | Kali Linux, Metasploit, Meterpreter |
| SOAR and threat intelligence | Tines, AbuseIPDB, VirusTotal, Slack, Wazuh Active Response |
| Framework | MITRE ATT&CK |

## Lab Architecture

```text
Windows 11 + Sysmon ──┐
                      ├──► Wazuh SIEM ──► Dashboards / Custom Detections
Ubuntu Linux ─────────┘                         │
                                               ▼
                                       Tines SOAR Workflow
                                               │
                                   AI + Threat Intelligence
                                     (AbuseIPDB / VirusTotal)
                                               │
                                               ▼
                                       Slack Notification
                                               │
                                               ▼
                                        Analyst Approval
                                               │
                                               ▼
                                     Wazuh Active Response

Kali Linux / Metasploit → Controlled C2 simulation → Windows telemetry
```

## Project Modules

| Module | Focus | Key Accomplishments |
|---|---|---|
| **1. Deploy the Server** | SIEM deployment | Installed Wazuh on Ubuntu, enabled archive logging, and configured Filebeat and searchable archive data. |
| **2. Connect Agents + Sysmon** | Endpoint onboarding | Connected Windows 11 and Ubuntu agents; installed Sysmon and configured Windows Sysmon event collection. |
| **3. Generate + Read Telemetry** | SOC investigation | Investigated account events (`4720`, `4726`, `4732`), correlated SIDs, and analyzed failed Linux SSH authentication. |
| **4. Build Your SOC Dashboard** | Security visualization | Built panels for failed Windows logons (`4625`), account changes, and Linux authentication. |
| **5. FIM + Your First Detection** | Detection engineering | Configured real-time FIM and tested custom rule `100200` for Windows Guest account enablement (`4722`). |
| **6. Active Response** | Adversary simulation | Simulated a Meterpreter session and investigated `whoami`, `tasklist`, and `ipconfig` activity in Wazuh. |
| **7. Setting up C2** | Detection and response | Used a custom repeated-SSH-failure detection (`100101`) and Wazuh response capabilities in the lab. |
| **8. Setting Up SOAR with Tines** | Security automation | Forwarded alerts to Tines, enriched IOCs, notified Slack, and tested analyst-approved `firewall-drop` response. |

## Investigation and Detection Highlights

### Windows Account Monitoring

Generated account creation, deletion, and local administrator group changes, then searched Wazuh events to identify the initiating account and affected user. When the account name was missing from a group-membership event, I pivoted on its **Security Identifier (SID)** to identify the account.

| Event ID | Activity |
|---|---|
| `4625` | Failed Windows logon |
| `4720` | User account created |
| `4722` | User account enabled |
| `4726` | User account deleted |
| `4732` | Member added to a security-enabled local group |

### File Integrity Monitoring and Detection Engineering

Configured real-time **FIM** on Windows and Ubuntu and validated file modification and deletion events. Developed and tested a custom Wazuh rule to detect when the built-in Windows **Guest** account was enabled, including **MITRE ATT&CK T1078** mapping.

### Controlled C2 and Discovery Simulation

Simulated post-compromise activity using **Metasploit/Meterpreter** in an authorized lab. Analyzed Wazuh/Sysmon process telemetry and parent-child relationships for user, process, and network discovery commands.

### SOAR and Analyst-Approved Response

Built a **Tines SOAR** workflow that received Wazuh alerts, enriched indicators with **AbuseIPDB** and **VirusTotal**, sent findings to **Slack**, and required analyst approval before invoking Wazuh **Active Response** (`firewall-drop`).

## Skills Demonstrated

- **SIEM administration:** Wazuh installation, endpoint onboarding, archive logging, and event collection.
- **SOC investigation:** Windows/Linux log analysis, authentication monitoring, event correlation, and SID pivots.
- **Detection engineering:** Custom Wazuh rules, real-time FIM, MITRE ATT&CK mapping, and alert validation.
- **Threat analysis:** Simulated C2 activity, process lineage investigation, and discovery-event analysis.
- **Security automation:** SOAR workflows, threat-intelligence enrichment, analyst approval, and active response.

## Module Documentation

Each module has its own detailed walkthrough containing configuration steps, screenshots, commands, and validation results. Once the module files are uploaded to this repository, replace the titles below with links to their actual relative GitHub paths.

1. [**Deploy the Server**](https://github.com/shbelay/Wazuh_Challenge/blob/main/1_Deploy%20the%20Server.md)
2. [**Connect Agents + Sysmon**](https://github.com/shbelay/Wazuh_Challenge/blob/main/2_Connect%20Agents%20and%20Sysmon.md)
3. [**Generate + Read Telemetry**](https://github.com/shbelay/Wazuh_Challenge/blob/main/3_Generate%20and%20Read%20Telemetry.md)
4. [**Build Your SOC Dashboard**](https://github.com/shbelay/Wazuh_Challenge/blob/main/4_Build%20Your%20SOC%20Dashboard.md)
5. [**FIM + Your First Detection**](https://github.com/shbelay/Wazuh_Challenge/blob/main/5_FIM%20and%20Detection.md)
6. [**Active Response**](https://github.com/shbelay/Wazuh_Challenge/blob/main/6_Active%20Response.md)
7. [**Setting up C2**](https://github.com/shbelay/Wazuh_Challenge/blob/main/7_Setting%20Up%20a%20C2.md)
8. [**Setting Up SOAR with Tines**](https://github.com/shbelay/Wazuh_Challenge/blob/main/8_Setting_Up_SOAR.md)

## Project Outcome

Completed a practical SOC workflow spanning **SIEM deployment → telemetry collection → investigation → detection development → visualization → adversary simulation → enrichment → analyst-approved response**.
