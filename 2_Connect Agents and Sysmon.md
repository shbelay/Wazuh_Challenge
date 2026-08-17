# Wazuh Challenge - Part 2: Connect Agents + Sysmon

## Overview

In this module, I expanded my Wazuh SIEM lab by connecting both a
**Windows 11 endpoint** and an **Ubuntu Linux endpoint** to the Wazuh
server. I then installed and configured **Sysmon** to provide richer
endpoint telemetry for security monitoring and future investigations.

The screenshots document the deployment of the Wazuh agents,
verification of agent data in Wazuh, installation of Sysmon on Windows
and Linux, and configuration of the Windows Wazuh agent to collect the
Sysmon event channel.

------------------------------------------------------------------------

## Objectives

-   Deploy a Wazuh agent to a Windows 11 endpoint
-   Deploy a Wazuh agent to an Ubuntu Linux endpoint
-   Verify endpoint data is reaching the Wazuh SIEM
-   Install Sysmon on Windows
-   Apply a Sysmon configuration
-   Configure the Windows Wazuh agent to collect Sysmon events
-   Install Sysmon for Linux
-   Apply a Linux Sysmon configuration

------------------------------------------------------------------------

# 1. Deploy the Windows Wazuh Agent

From the Wazuh dashboard, I selected **Deploy new agent**.

<img width="635" height="462" alt="image" src="https://github.com/user-attachments/assets/1c7cecc7-a2ea-4f71-a540-5ebeacab8de1" />

For the Windows endpoint, I selected:

-   **Operating system:** Windows
-   **Package:** MSI 32/64 bits
-   **Server address:** Wazuh server IP address
-   **Agent name:** `Windows-11-Agent`

<img width="1870" height="421" alt="image" src="https://github.com/user-attachments/assets/980c8f7e-e8c3-4722-bcb7-1a2ba8438631" />

<img width="951" height="522" alt="image" src="https://github.com/user-attachments/assets/9be8a1c6-72bb-44b9-bd2e-f63156f40a09" />


Wazuh generated a PowerShell command containing the required manager address and agent name.

<img width="1856" height="350" alt="image" src="https://github.com/user-attachments/assets/05046125-175d-440d-9445-fbfb02461861" />


## Install the Windows Agent

I opened **PowerShell as Administrator** on the Windows 11 endpoint and
ran the generated installation command:

``` powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='<WAZUH_SERVER_IP>' WAZUH_AGENT_NAME='Windows-11-Agent'
```

<img width="1151" height="277" alt="image" src="https://github.com/user-attachments/assets/559ec26c-a860-4bb5-b37a-753a58bb54a4" />


This downloaded the Wazuh MSI package and silently installed the agent
while configuring it to communicate with my Wazuh manager.

## Start the Windows Agent

After installation, I started the Wazuh service:

``` powershell
NET START Wazuh
```

PowerShell confirmed that the Wazuh service started successfully.

<img width="317" height="197" alt="image" src="https://github.com/user-attachments/assets/e7bc13f6-7273-4fb1-adbd-c3a32ab1f90d" />

------------------------------------------------------------------------

# 2. Deploy the Ubuntu Wazuh Agent

I returned to **Deploy new agent** in Wazuh and selected the Linux
package for my Ubuntu endpoint.

<img width="521" height="147" alt="image" src="https://github.com/user-attachments/assets/8aac14da-f295-48af-a95e-4ab9701b7f2e" />

<img width="593" height="396" alt="image" src="https://github.com/user-attachments/assets/18b9d983-5b24-4c98-bd5e-bb1effa94c7d" />

I configured:

-   **Operating system:** Linux
-   **Package:** DEB amd64
-   **Server address:** Wazuh server IP address
-   **Agent name:** `Ubuntu-Agent`

<img width="1862" height="372" alt="image" src="https://github.com/user-attachments/assets/39607507-7778-460a-8d99-ae2b99cc18f5" />

<img width="950" height="623" alt="image" src="https://github.com/user-attachments/assets/2d6a969a-8607-4804-b704-07c184a1747f" />


Wazuh generated the installation command for the endpoint.

## Download and Install the Linux Agent

On the Ubuntu system, I ran:

``` bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.7-1_amd64.deb && \
sudo WAZUH_MANAGER='<WAZUH_SERVER_IP>' WAZUH_AGENT_NAME='Ubuntu-Agent' \
dpkg -i ./wazuh-agent_4.14.7-1_amd64.deb
```

## Enable and Start the Agent

I then reloaded systemd, enabled the Wazuh agent to start automatically,
and started the service:

``` bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

The `enable` command created the systemd service link for the Wazuh
agent.

<img width="1622" height="597" alt="image" src="https://github.com/user-attachments/assets/6287b0ac-cd8b-439f-a5ba-1098b09afa31" />

<img width="1163" height="347" alt="image" src="https://github.com/user-attachments/assets/026c1a13-1e9e-4440-b9fc-3fab0a596890" />

<img width="1175" height="112" alt="image" src="https://github.com/user-attachments/assets/873079f4-8436-4bf4-87c2-16e55d181442" />

------------------------------------------------------------------------

# 3. Verify Agent Data in Wazuh

After deploying the endpoints, I returned to the Wazuh dashboard.

I navigated to:

``` text
Explore
└── Discover
```

<img width="322" height="240" alt="image" src="https://github.com/user-attachments/assets/0dbe2fee-d557-44bc-b3e5-c4f77c935e54" />


Using the Wazuh archives index, I reviewed the `agent.name` field.

The dashboard showed events associated with my Wazuh server and the
newly connected endpoints, including:

-   `Ubuntu-Agent`
-   `Windows-11-Agent`

This confirmed that endpoint telemetry was reaching Wazuh.

<img width="387" height="241" alt="image" src="https://github.com/user-attachments/assets/ee344b3f-d40c-4c91-975c-bb3f729efb81" />

<img width="633" height="335" alt="image" src="https://github.com/user-attachments/assets/89e54538-1e24-44ca-9d49-95e8db9b894b" />

------------------------------------------------------------------------

# 4. Install Sysmon on Windows

To increase endpoint visibility, I installed **Microsoft Sysmon** on the Windows 11 endpoint.

Sysmon provides detailed system activity telemetry that can be useful for detecting and investigating activity such as:

-   Process creation
-   Network connections
-   File creation
-   Registry modifications
-   Driver and image loading
-   Process access
-   DNS activity

## Download Sysmon

I downloaded Sysmon from Microsoft Sysinternals:

`https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon`

<img width="995" height="566" alt="image" src="https://github.com/user-attachments/assets/5663f690-e1b5-4746-8fde-a4d4e9bd48e8" />


I also obtained a Sysmon configuration from the **Olaf Hartong Sysmon
Modular** project:

`https://github.com/olafhartong/sysmon-modular`

From the repository, I opened:

``` text
sysmonconfig.xml
```

I selected **Raw** and saved the XML configuration to my Windows system.

<img width="1303" height="397" alt="image" src="https://github.com/user-attachments/assets/58b43f77-e4be-4872-be10-635d3bff7095" />

<img width="1572" height="210" alt="image" src="https://github.com/user-attachments/assets/08df9d43-fa00-4653-a0ae-487a5bf79a4c" />

<img width="973" height="525" alt="image" src="https://github.com/user-attachments/assets/ef6cc089-1256-4f9f-beaf-fd7ec97c989a" />

------------------------------------------------------------------------

# 5. Extract Sysmon and Apply the Configuration

I extracted the downloaded Sysmon archive into a `Sysmon` directory.

The extracted files included the Sysmon executables, and I placed the
downloaded configuration file in the same working directory.

From an elevated PowerShell session, I navigated to the Sysmon directory and transferred the sysmonconfig file to the Sysmon directory:

``` powershell
cd C:\Users\<USERNAME>\Downloads\Sysmon
```

<img width="1185" height="670" alt="image" src="https://github.com/user-attachments/assets/442d0ef2-4856-4f6e-a870-b770f0b8e032" />

<img width="1660" height="668" alt="image" src="https://github.com/user-attachments/assets/8abe73cb-2068-4bf9-8046-f668b156f325" />

I verified the files:

``` powershell
dir
```

I then installed Sysmon using the downloaded configuration:

``` powershell
.\Sysmon.exe -i .\sysmonconfig.xml
```

During the first installation, I accepted the **Sysinternals Software
License Terms**.

The output confirmed that:

-   The configuration file was validated
-   Sysmon was installed
-   The Sysmon driver was installed
-   The Sysmon service was started

<img width="658" height="142" alt="image" src="https://github.com/user-attachments/assets/6df6c7b3-b23f-4eca-bdc1-d514375614ed" />

<img width="1352" height="721" alt="image" src="https://github.com/user-attachments/assets/66ea6100-97de-459a-8786-8e9970ad6c2b" />

------------------------------------------------------------------------

# 6. Configure Wazuh to Collect Windows Sysmon Events

Sysmon writes its Windows telemetry to the following event channel:

``` text
Microsoft-Windows-Sysmon/Operational
```

To send these events to Wazuh, I opened the Wazuh agent configuration
located under:

``` text
C:\Program Files (x86)\ossec-agent
```

<img width="1085" height="571" alt="image" src="https://github.com/user-attachments/assets/30db9ace-5098-468c-acbf-a65c52a9f870" />

<img width="1005" height="592" alt="image" src="https://github.com/user-attachments/assets/4183efb0-6282-4192-a363-11ef89ecd225" />

I opened:

``` text
ossec.conf
```

I added the following `<localfile>` configuration:

``` xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

<img width="662" height="608" alt="image" src="https://github.com/user-attachments/assets/5d123c6d-b109-4520-8a99-c4d387ee06fb" />

<img width="792" height="760" alt="image" src="https://github.com/user-attachments/assets/99b075c5-f75b-46b1-9bcf-5e4e65141ea7" />

This tells the Wazuh agent to monitor the **Sysmon Operational** Windows
Event Log channel and ingest its events using the Windows event channel
format.

## Restart the Wazuh Service

Because the Wazuh agent configuration was changed, I opened
**Services**, located the **Wazuh** service, and restarted it so the
updated `ossec.conf` configuration would take effect.

<img width="947" height="612" alt="image" src="https://github.com/user-attachments/assets/96e188ed-2a33-415e-9f24-bc423a736e2f" />

<img width="945" height="617" alt="image" src="https://github.com/user-attachments/assets/b4ebf42c-9e87-4412-96e1-508586549993" />

------------------------------------------------------------------------

# 7. Install Sysmon for Linux

I also installed **Sysmon for Linux** on the Ubuntu endpoint.

I referenced Microsoft's Sysmon for Linux project:

`https://github.com/microsoft/sysmonforlinux`

<img width="912" height="192" alt="image" src="https://github.com/user-attachments/assets/4f1b506a-7b3b-48a0-85e3-6737c6f2a758" />


## Add the Microsoft Package Repository

I downloaded and installed Microsoft's Ubuntu package repository
configuration:

``` bash
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
```

``` bash
sudo dpkg -i packages-microsoft-prod.deb
```

I then refreshed the package list:

``` bash
sudo apt-get update
```

<img width="1120" height="357" alt="image" src="https://github.com/user-attachments/assets/4ade94e8-0acd-4818-b7bd-fbfe1a8e37f2" />

<img width="1252" height="170" alt="image" src="https://github.com/user-attachments/assets/f9014f26-2ba7-46c6-9b71-666397b2a42e" />

<img width="778" height="452" alt="image" src="https://github.com/user-attachments/assets/1cecdfa7-9a78-492e-b307-dcd23be4079e" />

## Install Sysmon for Linux

I installed Sysmon with:

``` bash
sudo apt-get install sysmonforlinux
```

------------------------------------------------------------------------

# 8. Download and Apply a Linux Sysmon Configuration

For the Linux Sysmon configuration, I downloaded Microsoft's
`collect-all.xml` configuration from the MSTIC-Sysmon repository:

``` bash
wget https://raw.githubusercontent.com/microsoft/MSTIC-Sysmon/refs/heads/main/linux/configs/collect-all.xml
```
<img width="1653" height="226" alt="image" src="https://github.com/user-attachments/assets/5bbeb64c-9487-4b67-a377-dab7f1dff4a7" />



I then applied the configuration:

``` bash
sudo sysmon -i collect-all.xml
```

<img width="953" height="295" alt="image" src="https://github.com/user-attachments/assets/2522afa2-8497-4c4a-846e-6eb14bb08956" />


The terminal output showed the configuration loading successfully and
the Sysmon service being created and started.

------------------------------------------------------------------------

# Validation

At the end of this module, the lab contained:

  Component                               Status
  --------------------------------------- ------------------------------------
  Wazuh Server                            Deployed
  Windows 11 Wazuh Agent                  Connected
  Ubuntu Wazuh Agent                      Connected
  Windows Sysmon                          Installed
  Windows Sysmon Configuration            Applied
  Windows Sysmon Event Channel            Added to Wazuh agent configuration
  Sysmon for Linux                        Installed
  Linux `collect-all.xml` Configuration   Applied

------------------------------------------------------------------------

# Key Takeaways

This module expanded the Wazuh environment from a standalone SIEM server
into a small multi-platform monitoring environment.

I practiced:

-   Wazuh agent deployment and enrollment
-   Windows and Linux endpoint onboarding
-   PowerShell-based software deployment
-   Linux package installation and service management
-   Windows Event Log collection
-   Wazuh `ossec.conf` configuration
-   Sysmon deployment and configuration
-   Endpoint telemetry collection
-   SIEM data validation

Adding Sysmon provides richer endpoint telemetry than standard
operating-system logging alone, giving me additional data that can be
used later for **threat hunting, detection engineering, and incident
investigation**.

------------------------------------------------------------------------

## Project Progress

``` text
Wazuh SIEM
│
├── Wazuh Server
│
├── Windows 11 Agent
│   └── Sysmon
│       └── Microsoft-Windows-Sysmon/Operational → Wazuh
│
└── Ubuntu Agent
    └── Sysmon for Linux
```

**Next:** Use the connected endpoints and enhanced telemetry to
generate, detect, and investigate security activity in Wazuh.
