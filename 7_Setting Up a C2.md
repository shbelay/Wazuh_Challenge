# Wazuh Challenge -- Setting Up C2 + Detecting Discovery Activity

## Overview

In this module, I simulated **Command-and-Control (C2)** activity in Wazuh lab using **Metasploit Framework** and a Windows test
endpoint.

I generated a Windows test payload from Kali Linux, transferred it to
the Windows endpoint, configured a Metasploit handler, and executed the
payload to establish a **Meterpreter session** back to Kali. I then
generated discovery activity and used **Wazuh/Sysmon telemetry**
to investigate the resulting process activity and MITRE ATT&CK context.

------------------------------------------------------------------------

## Objectives

-   Start Metasploit Framework on Kali Linux
-   Generate a Windows test payload
-   Configure a Metasploit handler
-   Transfer and execute the payload on the Windows lab endpoint
-   Establish a Meterpreter session
-   Generate system and process discovery activity
-   Review resulting telemetry in Wazuh
-   Analyze parent/child process relationships
-   Review MITRE ATT&CK context for discovery activity

------------------------------------------------------------------------

# 1. Start Metasploit

On Kali Linux, I opened a terminal and started:

``` bash
msfconsole
```
<img width="272" height="137" alt="image" src="https://github.com/user-attachments/assets/977bfbca-9948-43b3-942c-9b85589743f8" />

I opened a second terminal for payload generation.

------------------------------------------------------------------------

# 2. Generate the Windows Test Payload

Using `msfvenom`, I generated a Windows executable named:

``` text
belay_payload.exe
```
<img width="292" height="166" alt="image" src="https://github.com/user-attachments/assets/bc15ddf5-1563-46aa-89cb-ce82351fdc1c" />

<img width="910" height="317" alt="image" src="https://github.com/user-attachments/assets/32df6c60-857b-4c9d-8e5f-0e1931af7ac9" />

The payload was configured to establish a reverse connection from the
Windows lab endpoint to my Kali system.

------------------------------------------------------------------------

# 3. Configure the C2 Handler

Inside Metasploit, I configured a **multi/handler** to receive the
connection from the Windows endpoint and started the handler.

``` text
Windows Lab Endpoint
        │
        │ executes test payload
        ▼
Reverse Connection
        │
        ▼
Kali Linux
        │
        ▼
Metasploit Handler
        │
        ▼
Meterpreter Session
```

<img width="557" height="202" alt="image" src="https://github.com/user-attachments/assets/75337773-1d90-418f-ac6c-38bfca8adb1b" />

------------------------------------------------------------------------

# 4. Transfer the Payload

I uploaded `belay_payload.exe` to the temporary file-hosting service
shown in the lab and obtained a temporary download link.

<img width="1036" height="432" alt="image" src="https://github.com/user-attachments/assets/a2b165ce-8e94-4236-bf7a-c47940a6c784" />

I opened that link from the Windows test endpoint and downloaded:

``` text
belay_payload.exe
```

<img width="537" height="202" alt="image" src="https://github.com/user-attachments/assets/0b73674d-927a-4e9a-9861-75083ba981e5" />

The browser displayed a security warning.

<img width="1392" height="492" alt="image" src="https://github.com/user-attachments/assets/d01d446c-a912-4769-8e43-180de2712106" />

<img width="525" height="410" alt="image" src="https://github.com/user-attachments/assets/cf09ac4d-fa66-4dae-9519-898e0e956370" />

------------------------------------------------------------------------

# 5. Execute the Payload and Establish the Session

With the Metasploit handler waiting on Kali, I executed
`belay_payload.exe` on Windows.

<img width="807" height="476" alt="image" src="https://github.com/user-attachments/assets/1f9059dc-f571-4d53-9c8a-c72b42e1714f" />

<img width="476" height="91" alt="image" src="https://github.com/user-attachments/assets/1591787f-c834-458d-9acf-790c5a622c93" />

<img width="682" height="382" alt="image" src="https://github.com/user-attachments/assets/1bd99999-7d63-41ce-9a33-c122e2efbfc7" />

------------------------------------------------------------------------

# 6. Generate Discovery Activity

From the session, I generated several common Windows discovery
activities.

<img width="1058" height="245" alt="image" src="https://github.com/user-attachments/assets/e7bf0f68-b400-44cc-b262-2e8978477b37" />

The screenshots show:

``` text
tasklist
ipconfig
whoami
```

  Command      Information Collected
  ------------ ------------------------------
  `tasklist`   Running processes
  `ipconfig`   Network and IP configuration
  `whoami`     Current user/account context

I also entered a Windows command shell from Meterpreter before executing
some of the discovery commands.

<img width="537" height="126" alt="image" src="https://github.com/user-attachments/assets/fcf97db8-14bc-4f78-aaa4-b372ae49ca55" />

<img width="582" height="462" alt="image" src="https://github.com/user-attachments/assets/3d067c96-cdb2-4329-9a41-0a6aa47735da" />

<img width="752" height="284" alt="image" src="https://github.com/user-attachments/assets/d2091bd8-c359-4be4-97ee-657656a01f20" />

------------------------------------------------------------------------

# 7. Review the Activity in Wazuh

I returned to the Wazuh dashboard and reviewed the telemetry generated
by the Windows endpoint.

The screenshots show the discovery activity logged in Wazuh along with
detection and MITRE ATT&CK context.

``` text
C2 Session
     │
     ▼
Discovery Command
     │
     ▼
Windows Process Creation
     │
     ▼
Sysmon / Windows Telemetry
     │
     ▼
Wazuh Agent
     │
     ▼
Wazuh Detection
     │
     ▼
MITRE ATT&CK Context
```

<img width="675" height="522" alt="image" src="https://github.com/user-attachments/assets/58702a1a-5122-4f3c-917d-f32485837aca" />

<img width="1787" height="865" alt="image" src="https://github.com/user-attachments/assets/22c5349d-5479-4737-b7f4-41b6eda27445" />

------------------------------------------------------------------------

# 8. Investigate `whoami`

One captured event corresponded to:

``` cmd
whoami
```

<img width="1792" height="422" alt="image" src="https://github.com/user-attachments/assets/4eb07363-2ba7-4f38-8cf7-4e5541b8e3fb" />

The Wazuh telemetry identified the behavior as **User Discovery** and
exposed process information including the executable and its parent
process.

The process relationship observed was consistent with:

``` text
cmd.exe
   │
   └── whoami.exe
          │
          ▼
      User Discovery
```

------------------------------------------------------------------------

# 9. Investigate `tasklist`

I reviewed the telemetry associated with:

``` cmd
tasklist
```

The event showed:

``` text
Image: C:\Windows\System32\tasklist.exe
ParentImage: C:\Windows\System32\cmd.exe
```

Wazuh associated the behavior with **Process Discovery**.

``` text
cmd.exe
   │
   └── tasklist.exe
            │
            ▼
       Process Discovery
```

<img width="1356" height="472" alt="image" src="https://github.com/user-attachments/assets/0c17c01e-32ed-4710-b368-1f7cdffcb693" />

------------------------------------------------------------------------

# 10. Investigate `ipconfig`

The final discovery activity shown in the screenshots involved:

``` cmd
ipconfig
```

The telemetry showed:

``` text
Image: C:\Windows\System32\ipconfig.exe
ParentImage: C:\Windows\System32\cmd.exe
```

<img width="1347" height="472" alt="image" src="https://github.com/user-attachments/assets/e22cecc7-9de5-44ac-88ac-8a6ee5709bc1" />


------------------------------------------------------------------------

# 11. Discovery Activity Summary

| Activity | Windows Process | Detection Context |
|---|---|---|
| Identify current user | `whoami.exe` | User Discovery |
| Enumerate processes | `tasklist.exe` | Process Discovery |
| Enumerate network configuration | `ipconfig.exe` | System Network Configuration Discovery |


------------------------------------------------------------------------

# 12. Process Lineage

The telemetry demonstrated why **parent-child process relationships**
are valuable during SOC investigations.

``` text
belay_payload.exe
        │
        ▼
Meterpreter / C2 Activity
        │
        ▼
cmd.exe
        │
        ├── whoami.exe
        ├── tasklist.exe
        └── ipconfig.exe
```

Reviewing process lineage can help distinguish routine administrative
activity from commands launched as part of suspicious post-exploitation
behavior.

------------------------------------------------------------------------

# 13. MITRE ATT&CK Context

The screenshots show Wazuh providing ATT&CK-related context for the
generated discovery behaviors.

| Discovery Behavior | Purpose |
|---|---|
| **User Discovery** | Identify the current user/account context |
| **Process Discovery** | Enumerate running processes |
| **System Network Configuration Discovery** | Gather network configuration information |

This enrichment helps translate raw process telemetry into the likely
purpose of the observed activity.

------------------------------------------------------------------------

# SOC Investigation Perspective

Commands such as `whoami`, `tasklist`, and `ipconfig` are normal Windows
utilities. A SOC analyst should therefore avoid treating any one of them
as proof of malicious activity.

In this lab, the broader sequence provided the important context:

``` text
Payload Download
       │
       ▼
Payload Execution
       │
       ▼
C2 Session Established
       │
       ▼
Command Shell Activity
       │
       ├── User Discovery
       ├── Process Discovery
       └── Network Configuration Discovery
       │
       ▼
Wazuh Telemetry + ATT&CK Context
```

The correlation of multiple events in a short sequence creates a much
stronger indicator of post-compromise reconnaissance than any individual
discovery command alone.

------------------------------------------------------------------------


# Project Progress

``` text
Wazuh Challenge
│
├── Part 1 – Deploy the Server
├── Part 2 – Connect Agents + Sysmon
├── Part 3 – Generate + Read Telemetry
├── Part 4 – Build Your SOC Dashboard
├── Part 5 – FIM + Your First Detection
├── Part 6 – Active Response
│
└── Setting Up C2
    ├── Kali Linux
    │   ├── Metasploit
    │   ├── Test Payload
    │   └── Handler
    ├── Windows 11 Endpoint
    │   ├── Payload Download
    │   ├── Payload Execution
    │   └── Meterpreter Session
    └── Wazuh Investigation
        ├── User Discovery
        ├── Process Discovery
        ├── Network Configuration Discovery
        └── Process Lineage Analysis
```
