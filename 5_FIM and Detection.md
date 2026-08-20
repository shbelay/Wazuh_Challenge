# Wazuh Challenge - Part 5: FIM + Your First Detection

## Overview

In this module, I configured **File Integrity Monitoring (FIM)** so that Wazuh alerts the moment a monitored file is added, changed, or deleted. I then went a step further and wrote my own **custom detection rule** to alert when the built-in Guest account gets enabled on a Windows endpoint.

The work was split across two agents:

- **Windows 11 Agent** – real-time FIM on a new `Company Data` directory containing sensitive payroll data
- **Ubuntu Agent** – real-time FIM on a new `/opt/company-data` directory

I then created a custom rule in `local_rules.xml` to detect Windows Event ID 4722 (a user account was enabled) specifically for the Guest account, and validated the alert end-to-end.

---

## Objectives

- Configure file integrity monitoring in `ossec.conf` on the Windows agent
- Enable real-time monitoring for a custom directory (`Company Data`)
- Restart the Wazuh service to apply configuration changes
- Verify FIM events in the Wazuh dashboard (modified and deleted files)
- Configure real-time FIM on the Ubuntu agent for `/opt/company-data`
- Verify FIM events on the Ubuntu agent
- Identify the Windows Event ID for account enablement (4722)
- Write and test a custom detection rule for the Guest account being enabled
- Reload the Wazuh ruleset and validate the alert with a full log review

---

# 1. Create a Test Directory and File (Windows)

I RDP'd into my Windows virtual machine and created a new directory called `Company Data`.

<img width="160" height="137" alt="image" src="https://github.com/user-attachments/assets/03b89329-1e83-4d2e-bc2a-8dec23cab6ec" />

Inside it, I created a file, `payroll.txt`, and added sensitive content to simulate real business data:

<img width="846" height="241" alt="image" src="https://github.com/user-attachments/assets/f8b3bbd0-1457-4092-b0f5-d053354258c4" />

```
sensitive payroll data. do not edit!
```

I will use this file to monitor and later modify/delete to generate FIM events.

<img width="922" height="183" alt="image" src="https://github.com/user-attachments/assets/d83565c2-f096-41b5-8104-1778b49120a2" />

---

# 2. Configure File Integrity Monitoring (ossec.conf)

I opened `ossec.conf` to configure the FIM (`syscheck`) settings.

By default:

- FIM is **not disabled**
- The `frequency` that syscheck runs is every **12 hours** (43,200 seconds)
- A default set of files/directories is monitored, including several 32-bit programs
- One directory block already has `realtime="yes"` set — the Startup directory — meaning any file dropped there triggers an immediate event rather than waiting for the next scheduled scan

<img width="869" height="978" alt="image" src="https://github.com/user-attachments/assets/17aa98d1-3a32-44f7-806d-4cca1fbac9c9" />

Following that pattern, I copied the real-time directory block and pointed it at my new company data folder:

```xml
<directories realtime="yes">%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup</directories>
<directories realtime="yes">C:\Users\shbelay\Desktop\Company Data</directories>
```

<img width="895" height="65" alt="image" src="https://github.com/user-attachments/assets/a2645106-bbef-4768-8627-14b85bd2360d" />

Saving this meant Wazuh would monitor `Company Data` continuously rather than waiting for the next scheduled `syscheck` run.

---

# 3. Restart the Wazuh Service

Whenever `ossec.conf` is modified, the Wazuh service has to be restarted for the change to take effect.

```
Search
└── Services
    └── Wazuh
        └── Restart
```

I opened **Services**, located **Wazuh** (Wazuh Windows Agent), and selected **Restart**.


<img width="1012" height="507" alt="image" src="https://github.com/user-attachments/assets/37ff2596-5dfe-4381-a30c-d1fda37fdc8c" />

<img width="711" height="410" alt="image" src="https://github.com/user-attachments/assets/a53d3923-ffe1-4de2-b8d8-d846f862dc4e" />


---

# 4. Verify Real-Time FIM on the Wazuh Dashboard (Windows)

Back on the Wazuh dashboard, I confirmed both agents were active:

```
Agents Summary
├── Active (2)
└── Disconnected (0)
```

<img width="480" height="242" alt="image" src="https://github.com/user-attachments/assets/5448e914-31c9-480d-8dba-1f3f8d317cee" />

I selected my **Windows-11-Agent** and navigated to:

<img width="521" height="256" alt="image" src="https://github.com/user-attachments/assets/ea9e564a-c9b0-4203-8e33-2a17c7525186" />

```
File Integrity Monitoring
└── Events
```

<img width="952" height="164" alt="image" src="https://github.com/user-attachments/assets/f3f10d21-e6ea-40c4-a0bd-6c05cad5a015" />


At this point, there was nothing to see — no baseline event had fired yet, so I needed to force one.

<img width="687" height="232" alt="image" src="https://github.com/user-attachments/assets/9281d40a-fe57-48a4-947e-254e91444f9a" />


---

# 5. Trigger a File Modification Event

Back on the Windows VM, I opened `payroll.txt` under `Company Data`, added a line reading `testing`, and saved the file.

<img width="338" height="147" alt="image" src="https://github.com/user-attachments/assets/0ce84d68-ee09-4455-abc0-a6d3f70055be" />

Refreshing the Wazuh dashboard, an event appeared:

<img width="476" height="173" alt="image" src="https://github.com/user-attachments/assets/7ac26dce-3d4f-4379-a9a6-cc947ba019bb" />

| Field | Value |
|---|---|
| syscheck.path | `c:\users\shbelay\desktop\company data\payroll.txt.txt` |
| syscheck.event | modified |
| rule.description | Integrity checksum changed. |
| rule.level | 7 |
| rule.id | 550 |

The "integrity checksum changed" description is essentially Wazuh reporting a **file hash change** — confirmation that real-time FIM was working against the `Company Data` directory.

<img width="1907" height="568" alt="image" src="https://github.com/user-attachments/assets/92966524-dcb5-461c-bb9a-92cb3e553076" />

---

# 6. Trigger a File Deletion Event

Next, I tested deletion. I right-clicked `payroll.txt` in File Explorer and selected **Delete**. The folder was confirmed empty afterward.

<img width="705" height="623" alt="image" src="https://github.com/user-attachments/assets/5a665972-e710-4c5d-af4b-51a400306aac" />

<img width="847" height="280" alt="image" src="https://github.com/user-attachments/assets/a3dc18d5-34b6-4d7e-abdd-d38aafe53773" />

Refreshing the dashboard produced a second event:

| Field | Value |
|---|---|
| syscheck.path | `c:\users\shbelay\desktop\company data\payroll.txt.txt` |
| syscheck.event | deleted |
| rule.description | File deleted. |
| rule.level | 7 |
| rule.id | 553 |

Both the **modified** (rule 550) and **deleted** (rule 553) events confirmed real-time FIM was correctly capturing file lifecycle changes.

<img width="1910" height="128" alt="image" src="https://github.com/user-attachments/assets/50fa8367-d3e2-4249-a07f-aca16377fabe" />

**Reference:** [Wazuh Real-time Monitoring Documentation](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/basic-settings.html#real-time-monitoring)

---

# 7. Configure FIM on the Ubuntu Agent

I switched over to my Ubuntu server, which I was SSH'd into from my host.

I created a matching test directory and file:

```bash
mkdir /opt/company-data
echo "payroll data. do not edit!" > /opt/company-data/test.txt
```

<img width="702" height="130" alt="image" src="https://github.com/user-attachments/assets/1c629da8-430b-4c17-8021-266877cdf35f" />

I then opened the agent's configuration file:

```bash
nano /var/ossec/etc/ossec.conf
```

<img width="482" height="82" alt="image" src="https://github.com/user-attachments/assets/d8ebb0b4-9a8c-4711-aaee-63e82e97cdf0" />

Scrolling down to the `syscheck` block, I added a real-time directory entry pointed at `/opt/company-data`, following the same pattern used on the Windows agent:

```xml

  <directories realtime="yes">/opt/company-data</directories>

```

<img width="587" height="360" alt="image" src="https://github.com/user-attachments/assets/8d142d5c-e881-4218-911d-16ec4b788927" />

---

# 8. Verify FIM on Ubuntu Agent

Back on the Wazuh dashboard, I selected **Agents** → **Ubuntu-Agent**, then navigated to:

<img width="467" height="287" alt="image" src="https://github.com/user-attachments/assets/51a93013-b3f5-40be-bfac-b71b3654df26" />

<img width="552" height="312" alt="image" src="https://github.com/user-attachments/assets/1fc59788-08ae-44fb-a566-f8392da12a7e" />

```
File Integrity Monitoring
└── Events
```

Within the last six hours, nothing appeared yet — as expected, since the config change hadn't taken effect.

<img width="688" height="256" alt="image" src="https://github.com/user-attachments/assets/7624936a-acdc-4c2d-b48f-0b6cfd7a4ab3" />

Because I was on the Ubuntu VM, the last step was to restart the **Wazuh agent** service rather than use the Windows Services app:

```bash
sudo systemctl restart wazuh-agent
```

<img width="477" height="100" alt="image" src="https://github.com/user-attachments/assets/113ff11c-3d68-4de9-849e-485c5896ee17" />

I then went back into `test.txt` and added a second line, `testing`, to force a change:

```
payroll data. do not edit!
testing
```

<img width="221" height="97" alt="image" src="https://github.com/user-attachments/assets/dc59052c-b98f-4ff0-ad62-2e49de3b698b" />

Refreshing the dashboard, an event appeared confirming real-time FIM was active on the Linux side as well:

| Field | Value |
|---|---|
| syscheck.path | `/opt/company-data/test.txt` |
| syscheck.event | modified |
| rule.description | Integrity checksum changed. |
| rule.level | 7 |
| rule.id | 550 |

<img width="1905" height="117" alt="image" src="https://github.com/user-attachments/assets/55b1eddf-37b0-423f-98e6-36b58f6b68f2" />

---

# 9. Prepare a Custom Detection: Enable the Guest Account

With FIM validated on both agents, I moved to the second half of the module — building a **custom detection rule**.

I chose to detect the Windows built-in **Guest** account being enabled, since it starts disabled by default and enabling it is a common local-persistence technique.

On the Windows VM, I went into **Computer Management** → **Local Users and Groups** → **Users**, opened the Guest account's **Properties**, and unchecked **Account is disabled**.

Confirming afterward showed the Guest account no longer had the disabled-account indicator (the down-arrow icon) next to it — it was now enabled.

<img width="685" height="351" alt="image" src="https://github.com/user-attachments/assets/9009e870-47f4-4c28-b149-35fef05c14eb" />

<img width="1010" height="723" alt="image" src="https://github.com/user-attachments/assets/2e3eebbb-983a-4e1b-a354-8421efdfd6c3" />

<img width="1007" height="366" alt="image" src="https://github.com/user-attachments/assets/b1cbedf4-ec1c-4e5b-a17f-89949689724a" />

---

# 10. Test Query for Event ID 4722

Before writing the rule, I ran a test query in the Wazuh dashboard to confirm what the raw event looked like.

I searched for the Guest account combined with Windows Event ID **4722** (a user account was enabled) and got a single hit.

I pulled the two field names I needed for the rule query:

```
data.win.eventdata.targetUserName: Guest AND data.win.system.eventID: 4722
```

<img width="870" height="835" alt="image" src="https://github.com/user-attachments/assets/3191cfd5-297a-43e8-914d-c378f5fb6b7f" />


The returned event confirmed the relevant fields:

| Field | Value |
|---|---|
| data.win.eventdata.targetUserName | Guest |
| data.win.system.eventID | 4722 |
| data.win.system.message | "A user account was enabled." |

---

# 11. Build the Custom Rule

I navigated to:

```
☰ Menu
└── Server management
    └── Rules
        └── Custom rules
```

<img width="704" height="156" alt="image" src="https://github.com/user-attachments/assets/1284559a-a58b-4bd4-afc4-16ec51818e8e" />

and opened **local_rules.xml**, which already contained one existing custom rule (`100001` — sshd authentication failed from an IP).

<img width="1877" height="112" alt="image" src="https://github.com/user-attachments/assets/666048dd-ca11-455d-ae4e-d11edc592934" />

I wrote a new rule targeting the Guest account being enabled:

```xml
<group name="windows,windows_security,account_changed,adduser">

  <rule id="100200" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4722$</field>
    <field name="win.eventdata.targetUserName">^Guest$</field>
    <description>Windows Guest account was enabled.</description>
    <mitre>
      <id>T1078</id>
    </mitre>
    <group>
      windows,
      windows_account_management,
      account_enabled,
      guest_account,
    </group>
  </rule>

</group>
```

<img width="1887" height="824" alt="image" src="https://github.com/user-attachments/assets/873bb1e3-1dea-4a0a-aab4-d23331e8bc71" />

**Rule logic:**
- `if_sid 60103` — matches on the parent Windows Security event rule
- `win.system.eventID = 4722` — restricts to account-enabled events
- `win.eventdata.targetUserName = Guest` — restricts specifically to the Guest account
- `level 12` — a high-severity alert, appropriate for a normally-disabled built-in account being activated
- Mapped to **MITRE ATT&CK T1078 – Valid Accounts**

---

# 12. Add the Rule to local_rules.xml and Reload

I pasted the rule into `local_rules.xml` in the Wazuh rules editor and selected **Save**.

Wazuh flagged that the changes wouldn't take effect until a reload was performed, so I selected **Reload** to apply the new rule to the ruleset.

<img width="1882" height="125" alt="image" src="https://github.com/user-attachments/assets/94f335f4-8098-403b-a4a9-01bfedb997e9" />

---

# 13. Trigger and Verify the Custom Detection Alert

To test the new rule, I went back to the Windows VM and toggled the Guest account — disabling it and re-enabling it via **Computer Management** → **Guest Properties** → unchecking **Account is disabled** → **Apply/OK**.

<img width="1020" height="732" alt="image" src="https://github.com/user-attachments/assets/cda235eb-a034-40b6-815b-354cb3fb8cf3" />

Back in the Wazuh dashboard, searching the same query returned **2 hits** this time:

```
data.win.eventdata.targetUserName: Guest AND data.win.system.eventID: 4722
```

| Field | Value |
|---|---|
| data.win.eventdata.targetUserName | Guest |
| data.win.system.eventID | 4722 |
| data.win.system.message | "A user account was enabled." |

<img width="998" height="828" alt="image" src="https://github.com/user-attachments/assets/6a768b15-16ff-499d-b7f0-823d3852d96a" />


Expanding the full log confirmed the custom rule fired correctly:

| Field | Value |
|---|---|
| rule.description | Windows Guest account was enabled. |
| rule.id | 100200 |
| rule.level | 12 |
| rule.mail | true |
| rule.mitre.id | T1078 |
| rule.mitre.tactic | Defense Evasion, Persistence, Privilege Escalation, Initial Access |
| rule.mitre.technique | Valid Accounts |
| rule.groups | windows, windows_security, account_changed, adduser, windows, windows_account_management, account_enabled, guest_account |
| agent.name | Windows-11-Agent |
| decoder.name | windows_eventchannel |
| data.win.eventdata.subjectUserName | shbelay |
| data.win.eventdata.targetUserName | Guest |
| data.win.eventdata.targetDomainName | Wazuh-Windows11 |

The alert triggered exactly as intended, and the `rule.mail: true` flag confirmed it was configured to notify — meaning enabling the Guest account on this host now generates a high-severity, actionable alert instead of going unnoticed.

<img width="1794" height="1733" alt="image" src="https://github.com/user-attachments/assets/33697f8b-a258-4185-818d-cefe77c73b0e" />

---

# Key Takeaways

In this module, I practiced:

- Configuring Wazuh's `syscheck` (FIM) module in `ossec.conf`
- Enabling real-time monitoring for a custom directory on both Windows and Linux agents
- Understanding the default FIM schedule (12-hour scans) versus real-time monitoring
- Restarting the Wazuh agent service (via Windows Services and via `systemctl` on Linux) to apply configuration changes
- Validating FIM by generating and reviewing both **file modified** and **file deleted** events
- Using the dashboard to test-query raw Windows Security events before writing detection logic
- Writing a custom Wazuh detection rule in `local_rules.xml`
- Mapping a custom rule to MITRE ATT&CK (T1078 – Valid Accounts)
- Reloading the Wazuh ruleset for changes to take effect
- Validating an end-to-end custom detection, from trigger to full alert log

A key takeaway from this module was that FIM alone tells you *something* changed, but a **custom rule** is what turns a raw, noisy event (like "a user account was enabled") into a **meaningful, actionable, high-severity alert** tied to a specific security concern — in this case, the sudden activation of a normally-disabled built-in account.

---

# Project Progress

```
Wazuh Challenge
│
├── Part 1 – Deploy the Server
│   ├── Install Wazuh
│   ├── Enable archive logging
│   └── Configure archive indexing
│
├── Part 2 – Connect Agents + Sysmon
│   ├── Windows 11 Agent
│   │   └── Sysmon
│   └── Ubuntu Agent
│       └── Sysmon for Linux
│
├── Part 3 – Generate + Read Telemetry
│   ├── Windows Reconnaissance
│   ├── Account Management Events
│   ├── SID Correlation
│   └── Linux SSH Authentication Analysis
│
├── Part 4 – Build Your SOC Dashboard
│   ├── Failed Windows Logon
│   ├── Windows Account Changes
│   └── Linux Authentication Activity
│
└── Part 5 – FIM + Your First Detection
    ├── Real-Time FIM (Windows – Company Data)
    ├── Real-Time FIM (Ubuntu – /opt/company-data)
    └── Custom Rule: Guest Account Enabled (Rule 100200 / T1078)
```

---

# Outcome

I successfully configured real-time File Integrity Monitoring on both my Windows and Ubuntu agents, validating that Wazuh could detect file creation, modification, and deletion the moment they occurred rather than waiting on the default 12-hour scan cycle.

Building on that, I wrote and deployed my first custom detection rule — one that watches for the Guest account being enabled and fires a high-severity, MITRE-mapped alert. This module moved the project from **passively collecting telemetry** to **actively writing detection logic**, which is the core skill of turning a SIEM from a log repository into a working detection platform.
