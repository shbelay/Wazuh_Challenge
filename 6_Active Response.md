# Wazuh Challenge - Part 6: Active Response

## Overview

In this module, I configured **Active Response** so that Wazuh doesn't just alert on suspicious activity — it automatically reacts to it. I built on the custom SSH brute-force detection rule from earlier work and wired it to a **firewall-drop** active response, so that repeated failed SSH logons from the same source IP now get blocked automatically on the Ubuntu agent, without any manual intervention.

The work was done entirely on the **Ubuntu Agent** and the **Wazuh Manager**, and validated end-to-end with a live SSH brute-force simulation from the Windows host.

---

## Objectives

- Simulate failed SSH authentication attempts against the Ubuntu agent
- Write a custom correlation rule to detect multiple SSH login failures from the same source IP within a defined timeframe
- Save and reload the custom ruleset
- Configure an `active-response` block in `ossec.conf` on the Wazuh Manager
- Bind the `firewall-drop` command to the new custom rule ID
- Restart the Wazuh Manager to apply the configuration
- Verify the active response script is registered and running on the agent
- Validate the full detection-to-response pipeline with a live test (ping + repeated failed SSH)

---

# 1. Generate Failed SSH Login Attempts

I SSH'd into my Ubuntu virtual machine from a Windows PowerShell session and intentionally entered the wrong password three times in a row to generate `Permission denied` events.

```
ssh shbelay@20.*.*.*
Permission denied, please try again.
Permission denied, please try again.
Permission denied (publickey,password).
```

<img width="839" height="419" alt="image" src="https://github.com/user-attachments/assets/20e01735-5cfe-493e-ad98-e69fd6141f3c" />

---

# 2. Create a Custom Correlation Rule

The goal was a rule that fires when there are **three failed SSH logons within a two-minute window** from the same source IP — a classic brute-force indicator.

I navigated to:

```
☰ Menu
└── Server management
    └── Rules
        └── Custom rules
```

<img width="337" height="900" alt="image" src="https://github.com/user-attachments/assets/52580797-e13a-41f0-8786-72ddeba87ff6" />

<img width="627" height="227" alt="image" src="https://github.com/user-attachments/assets/8de6cb53-25ed-4f16-af4b-fae1cf072bdf" />

and opened **local_rules.xml**, which already contained one existing custom rule (`100001` — sshd authentication failed from a specific IP).

<img width="1890" height="443" alt="image" src="https://github.com/user-attachments/assets/4ee94800-f2aa-4cc4-ba26-aa46ccac1d07" />

With the help of ChatGPT, I pasted following correlation rule:

```xml
<group name="local,syslog,sshd,authentication_failed,">

  <rule id="100101" level="10" frequency="3" timeframe="120">
    <if_matched_sid>5760</if_matched_sid>
    <same_source_ip />
    <description>Multiple SSH login failures observed from the same source IP</description>
    <mitre>
      <id>T1110</id>
    </mitre>
    <group>authentication_failed,ssh_bruteforce,credential_access,</group>
  </rule>

</group>
```

<img width="1887" height="824" alt="image" src="https://github.com/user-attachments/assets/d335cb09-86b8-42d1-8fb1-e250b93a47c1" />

**Rule logic:**

- `if_matched_sid 5760` — triggers off the base "sshd authentication failed" rule
- `frequency="3" timeframe="120"` — requires 3 matches within 120 seconds
- `same_source_ip` — correlates the failures to a single source IP
- Mapped to **MITRE ATT&CK T1110 – Brute Force**

I clicked **Save**, and **Reload**.

<img width="1880" height="117" alt="image" src="https://github.com/user-attachments/assets/afe179f3-8830-406b-939d-c1a7681c891d" />

---

# 3. Configure Active Response (ossec.conf)

To let Wazuh perform an automated action in response to this rule, I SSH'd into the **Wazuh Manager** and opened the manager's configuration file:

```
sudo nano /var/ossec/etc/ossec.conf
```

<img width="556" height="102" alt="image" src="https://github.com/user-attachments/assets/57987db6-8e50-48ae-b4cd-517110f48995" />

I scrolled down to the configuration section related to active response. The block was present but commented out:

```xml
<!--
<active-response>
  active-response options here
</active-response>
-->
```
<img width="282" height="112" alt="image" src="https://github.com/user-attachments/assets/2f091957-fded-4dd7-8d4c-1f85a4220745" />

I removed the `<!-- -->` comment markers so Wazuh would actually parse the block, then filled it in:

```xml
<active-response>
  <disabled>no</disabled>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>100101</rules_id>
</active-response>
```
<img width="320" height="126" alt="image" src="https://github.com/user-attachments/assets/1f88baeb-6884-43b4-a1c4-88a6b9ae48c3" />

**Configuration breakdown:**

- `<disabled>no</disabled>` — enables the active response block
- `<command>firewall-drop</command>` — must match the name of a built-in/available active response script
- `<location>local</location>` — tells Wazuh to run the script on the **agent that generated the alert** (in this case, the Ubuntu server), rather than on the manager or on all agents
- `<rules_id>100101</rules_id>` — ties this response specifically to the custom SSH brute-force rule created in Step 2

The Rule ID is in the screenshot below
<img width="1794" height="1215" alt="image" src="https://github.com/user-attachments/assets/7388b9aa-9c19-4dfb-b41a-9421df8e0dcd" />

---

# 4. Restart the Wazuh Manager and Verify

With the configuration saved, I restarted the manager service for the change to take effect:

```
sudo systemctl restart wazuh-manager.service
```
<img width="660" height="97" alt="image" src="https://github.com/user-attachments/assets/8ef4ce6d-e34d-475a-9c1a-a56b6823fc77" />

I then verified which active responses were registered and running on the agent using:

```
sudo /var/ossec/bin/agent_control -L
```

The output confirmed the response was correctly registered:

```
Wazuh agent_control. Available active responses:
   Response name: firewall-drop0, command: firewall-drop
```
<img width="583" height="143" alt="image" src="https://github.com/user-attachments/assets/e9467d0f-e4fe-498d-bd45-2d48c167ec70" />

---

# 5. Proof of Concept — Trigger the Response

To validate the full pipeline, I set up a continuous ping from my Windows host to the Ubuntu machine:

```
ping 10.1.1.5 -t
```
<img width="447" height="641" alt="image" src="https://github.com/user-attachments/assets/ba2f8efd-d77c-4875-95f2-8be6798c7589" />

The ping ran cleanly with normal replies (`bytes=32 time=1ms TTL=64`), confirming baseline connectivity before the test.

On another PowerShell window, I then SSH'd into the Ubuntu machine and intentionally failed authentication three times:

```
ssh shbelay@10.1.1.5
Permission denied, please try again.
Permission denied, please try again.
Permission denied (publickey,password).
```
<img width="687" height="282" alt="image" src="https://github.com/user-attachments/assets/2125fc72-85e6-4f7c-9304-a5c39239645e" />

Almost immediately, the SSH connection dropped — confirming the source IP had been automatically added to the firewall drop list. Switching back to the ping window confirmed it too:

```
Reply from 10.1.1.5: bytes=32 time=1ms TTL=64
Request timed out.
```

<img width="454" height="642" alt="image" src="https://github.com/user-attachments/assets/ba902df4-cd81-4ef3-83f7-85564ecfa04e" />

Wazuh's active response had blocked traffic from my Windows host at the firewall level.

---

# 6. Verify in the Wazuh Dashboard

Back in the Wazuh dashboard, the alert timeline showed the full chain of events firing in sequence:

| Time | Rule Description |
|---|---|
| 23:50:36.109 | sshd: authentication failed. |
| 23:50:34.114 | PAM: User login failed. |
| **23:49:09.555** | **Host Blocked by firewall-drop Active Response** |
| 23:49:08.125 | syslog: User missed the password more than one time |
| 23:49:08.001 | sshd: connection reset |
| **23:49:08.079** | **Multiple SSH login failures observed from the same source IP** |

<img width="727" height="277" alt="image" src="https://github.com/user-attachments/assets/4b2e4d45-0808-46e2-9fbb-d1936ae4857e" />

This confirmed the end-to-end pipeline worked exactly as designed:

1. Three failed SSH attempts triggered the custom correlation rule (`100101`)
2. The correlation rule fired the bound active response (`firewall-drop`)
3. The offending IP was blocked at the firewall level on the Ubuntu agent
4. Subsequent connection attempts (and even ICMP pings) from that IP were dropped

---

# Key Takeaways

In this module, I practiced:

- Writing a **frequency/timeframe correlation rule** to detect repeated failed SSH logons from the same source IP
- Understanding `if_matched_sid`, `frequency`, `timeframe`, and `same_source_ip` as building blocks for brute-force detection
- Mapping a custom rule to MITRE ATT&CK (T1110 – Brute Force)
- Enabling and configuring the `<active-response>` block in `ossec.conf` on the Wazuh Manager
- Understanding the difference between `location: local` (agent that generated the alert) versus other placement options
- Binding an active response command to a specific custom rule ID via `<rules_id>`
- Verifying registered active responses on an agent using `agent_control -L`
- Validating a full detection-to-response loop with a live brute-force simulation and firewall block

The biggest shift in this module was moving from **detection** to **response**. Up to this point, Wazuh was purely observational — it told me *something happened*. With active response configured, Wazuh now *does something about it*: three bad SSH attempts from an IP is enough to get that IP automatically firewalled off the box, with zero analyst intervention required.

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
├── Part 5 – FIM + Your First Detection
│   ├── Real-Time FIM (Windows – Company Data)
│   ├── Real-Time FIM (Ubuntu – /opt/company-data)
│   └── Custom Rule: Guest Account Enabled (Rule 100200 / T1078)
│
└── Part 6 – Active Response
    ├── Custom Rule: SSH Brute Force (Rule 100101 / T1110)
    ├── Active Response: firewall-drop bound to Rule 100101
    └── Proof of Concept: Live brute-force → automatic IP block
```

---

# Outcome

I successfully closed the loop between detection and response by building a custom SSH brute-force correlation rule and binding it to Wazuh's `firewall-drop` active response. When my simulated attacker failed SSH authentication three times within two minutes, Wazuh automatically blocked that source IP at the firewall — confirmed both by the dropped SSH session and by pings from the same host silently timing out.

This module moved the project from **writing detection logic** to **automating remediation**, which is the difference between a SIEM that just tells you about a problem and a SIEM that helps shut it down.
