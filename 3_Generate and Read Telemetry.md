# Wazuh Challenge - Part 3: Generate + Read Telemetry

## Overview

In this module, I generated endpoint telemetry on my
**Windows 11** and **Ubuntu Linux** endpoints and then used **Wazuh** to
locate and interpret the resulting events.

The Windows portion focused on system/account discovery, local account
changes, and local administrator group membership. I then reviewed the
corresponding Windows Security events in Wazuh. The Linux portion
generated a failed SSH authentication attempt and used Wazuh telemetry
to investigate the attempted login.

------------------------------------------------------------------------

## Objectives

-   Generate Windows system and account discovery activity
-   Create, modify, and delete a local test account
-   Modify membership of the local Administrators group
-   Locate the resulting Windows Security events in Wazuh
-   Correlate Windows Event IDs with the actions performed
-   Pivot on a Security Identifier (SID) when a username is not directly
    displayed
-   Generate a failed SSH authentication attempt against Ubuntu
-   Locate and analyze the Linux authentication telemetry in Wazuh

------------------------------------------------------------------------

# 1. Generate Windows Reconnaissance Telemetry

I began by running several Windows commands useful for system and
account discovery.

## Identify the Current User and Host

``` cmd
whoami
```

This displayed the current host/user context.

<img width="282" height="47" alt="image" src="https://github.com/user-attachments/assets/d6902160-33b2-45ae-9543-00b4d4ba035e" />

## Enumerate Network Configuration

``` cmd
ipconfig /all
```

This displayed detailed network information including the IPv4 address,
subnet mask, default gateway, and adapter configuration.

<img width="847" height="565" alt="image" src="https://github.com/user-attachments/assets/334482d4-b9b2-42a0-bb21-33dd994aca68" />

## Enumerate Local Users

``` cmd
net user
```

This returned the local accounts configured on the Windows 11 endpoint.


<img width="752" height="173" alt="image" src="https://github.com/user-attachments/assets/cabd1428-0c03-4f3e-82d2-da908edfb1cd" />

## Enumerate Local Administrators

``` cmd
net localgroup administrators
```

This displayed the accounts with membership in the local
**Administrators** group.


<img width="845" height="197" alt="image" src="https://github.com/user-attachments/assets/fd7d3b5b-59f6-4158-9dbe-9f671d3d91d9" />


These commands also demonstrate the type of host and account discovery
activity that may be relevant during a security investigation.

------------------------------------------------------------------------

# 2. Generate Windows Account-Management Activity

Next, I generated account-management telemetry for later analysis in
Wazuh.

## Enable the Guest Account

``` cmd
net user guest /active:yes
```

## Change the Guest Password

``` cmd
net user guest <LAB_PASSWORD>
```

<img width="408" height="125" alt="image" src="https://github.com/user-attachments/assets/42220e80-223a-4448-9ce6-24806fcfd365" />

## Add the Test User to Administrators

``` cmd
net localgroup administrators belay /add
```

<img width="547" height="55" alt="image" src="https://github.com/user-attachments/assets/c880a333-7301-48d8-830f-f6f6bcdb46f9" />

I verified group membership with:

``` cmd
net localgroup administrators
```

The output showed `belay` in the Administrators group.

<img width="842" height="206" alt="image" src="https://github.com/user-attachments/assets/692192e2-c56f-44af-91d6-c7699ce3d071" />

## Create/Configure the Test Account

The screenshots also show:

``` cmd
net user belay <LAB_PASSWORD> /add
```

The `/add` parameter creates the local account if it does not already
exist. This generated account-creation telemetry that I later reviewed
in Wazuh.

<img width="448" height="57" alt="image" src="https://github.com/user-attachments/assets/4150a705-dc44-49d2-9bab-2b5f44d721fc" />

------------------------------------------------------------------------

# 3. Review the Generated Telemetry in Wazuh

After generating the Windows activity, I moved to the **Wazuh
dashboard** to locate and interpret the resulting events.

The goal was to determine:

-   Who performed the action?
-   Which account was affected?
-   Which endpoint generated the event?
-   What Windows Event ID represented the activity?
-   What additional fields could be used to correlate related activity?

------------------------------------------------------------------------

# 4. Investigate User Creation -- Event ID 4720

I located the Windows Security event associated with creation of the
`belay` account.

``` text
4720 – A user account was created
```

At the top of the commands logged, we'll see a log for Windows Event ID 4726. Based on Ultimate Windows Security at https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4720.

<img width="867" height="328" alt="image" src="https://github.com/user-attachments/assets/3087dd75-cc67-4cda-a5ba-0e6f043fa30c" />

The Wazuh event identified information such as:

-   **Initiating account:** `shbelay`
-   **Target account:** `belay`
-   **Endpoint:** `Wazuh-Windows11`
-   **Windows Event ID:** `4720`
-   **Wazuh rule description:** `User account was created or enabled`

This allowed me to determine **who performed the action, which account
was affected, and where it occurred**.

From a SOC perspective, unexpected account creation can be important
because new accounts may be associated with unauthorized access or
persistence.

<img width="1794" height="2124" alt="image" src="https://github.com/user-attachments/assets/a93a59f2-04d6-4df1-9c94-460cdb633ccd" />

------------------------------------------------------------------------

# 5. Generate and Investigate Account Deletion

After reviewing the creation telemetry, I deleted the test account:

``` cmd
net user belay /delete
```

<img width="382" height="60" alt="image" src="https://github.com/user-attachments/assets/e8d71380-bad6-4638-bf37-6a8392a17d10" />


I then returned to Wazuh and located the corresponding event.

``` text
4726 – A user account was deleted
```

<img width="865" height="300" alt="image" src="https://github.com/user-attachments/assets/37a62ee9-ae12-4dac-919e-7cb45e583739" />


The Wazuh telemetry identified `belay` as the target account and showed
the account deletion occurring on the Windows endpoint.

The associated Wazuh rule description identified the activity as:

``` text
User account disabled or deleted
```

<img width="1794" height="1819" alt="image" src="https://github.com/user-attachments/assets/b75393f4-e3a4-4387-9a19-d9b494626809" />


This demonstrated how Wazuh could expose both the creation and deletion
portions of the local account lifecycle.

------------------------------------------------------------------------

# 6. Investigate Administrator Group Modification -- Event ID 4732

I next investigated the action that added `belay` to the local
Administrators group.

The relevant Windows Security event was:

``` text
4732 – A member was added to a security-enabled local group
```

<img width="706" height="378" alt="image" src="https://github.com/user-attachments/assets/b6e08630-b030-46c3-852a-9e0c971a2e6c" />


After filtering for Event ID `4732`, the event showed that:

-   `shbelay` initiated the action
-   The affected group was `Administrators`
-   A member had been added to the group

However, the account name of the added member was not clearly populated
in the initial event view.

Instead, the event contained the member's **Security Identifier (SID)**.

<img width="961" height="343" alt="image" src="https://github.com/user-attachments/assets/f634c4c4-962b-4c7c-93a9-977a7a63bc93" />

------------------------------------------------------------------------

# 7. Pivot on the SID

To identify the account that had been added to Administrators, I copied
the SID from the Event ID 4732 telemetry and searched for it in Wazuh.

``` text
Event ID 4732
      │
      ├── Group: Administrators
      │
      └── Member SID
              │
              ▼
       Search SID in Wazuh
              │
              ▼
       targetUserName: belay
```

Related telemetry showed:

``` text
targetUserName: belay
```

<img width="898" height="866" alt="image" src="https://github.com/user-attachments/assets/c1de5e7e-1997-4c0d-98e1-43e263227424" />


This confirmed that `belay` was the account associated with the SID and
therefore the account added to the Administrators group.

## SOC Takeaway

This was a useful example of event correlation. Security logs do not
always provide the answer in a single field or event.

When the username was not directly available, I used another
identifier---the **SID**---to pivot into related telemetry and identify
the affected account.

------------------------------------------------------------------------

# 8. Generate Linux SSH Authentication Telemetry

I then generated authentication telemetry against my Ubuntu endpoint.

I attempted to SSH into the Ubuntu system using the fake username:

``` text
attacker
```

The purpose was to intentionally generate a failed SSH authentication
event that I could locate and analyze in Wazuh.

------------------------------------------------------------------------

# 9. Investigate the Failed SSH Attempt

I searched Wazuh for:

``` text
attacker
```

Wazuh returned events associated with the failed authentication attempt.

The telemetry exposed useful investigation fields including:

-   **Agent:** `Ubuntu-Agent`
-   **Username:** `attacker`
-   **Source IP:** originating system
-   **Source port:** originating connection port
-   **Geolocation information:** when available
-   **Decoder:** `sshd`
-   **Full log:** original SSH authentication message

The raw log showed an SSH authentication failure involving an invalid
user.

<img width="1120" height="823" alt="image" src="https://github.com/user-attachments/assets/6a975e5f-6644-44e3-8b39-d2740999c40a" />


This demonstrated how Wazuh parses Linux authentication logs into
searchable fields while retaining the original log for additional
context.

------------------------------------------------------------------------

# 10. Windows Event IDs Investigated

  | Event ID | Description | Lab Activity |
|---|---|---|
| **4720** | A user account was created | Created the `belay` test account |
| **4726** | A user account was deleted | Deleted the `belay` test account |
| **4732** | A member was added to a security-enabled local group | Added `belay` to the Administrators group |
                   
-----------------------------------------------------------------------

These events provide useful telemetry when investigating account
creation, account removal, and changes to privileged local groups.

------------------------------------------------------------------------

# 11. Investigation Workflow Practiced

``` text
Generate Activity
      │
      ▼
Endpoint Produces Logs
      │
      ▼
Wazuh Collects Telemetry
      │
      ▼
Search Relevant Events
      │
      ▼
Review Event ID + Fields
      │
      ▼
Identify User / Host / Action
      │
      ▼
Pivot on SID or Other Identifiers
      │
      ▼
Correlate Related Events
      │
      ▼
Determine What Happened
```

Rather than only confirming that logs existed, I practiced reading the
underlying telemetry and correlating multiple fields to reconstruct the
activity.

------------------------------------------------------------------------

# Key Takeaways

In this module, I practiced:

-   Windows system reconnaissance
-   Local user enumeration
-   Local administrator enumeration
-   Local account creation and deletion
-   Privileged group modification
-   Windows Security Event ID analysis
-   Wazuh event searching and filtering
-   SID-based investigation pivots
-   Event correlation
-   Linux SSH authentication analysis
-   Source IP and username identification
-   Reading parsed fields and raw log data

A major takeaway was that an analyst cannot always rely on a single
event field to provide the complete answer. The Event ID 4732
investigation required me to take the SID from one event and search
related telemetry to identify the account that had been added to
Administrators.

------------------------------------------------------------------------

# Project Progress

``` text
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
└── Part 3 – Generate + Read Telemetry
    ├── Windows Reconnaissance
    ├── Account Management
    │   ├── Event ID 4720 – Account Created
    │   ├── Event ID 4726 – Account Deleted
    │   └── Event ID 4732 – Member Added to Local Group
    ├── SID Correlation
    └── Linux SSH Authentication Analysis
```

------------------------------------------------------------------------

# Outcome

I successfully generated controlled Windows and Linux security activity
and used Wazuh to investigate the resulting telemetry.

I began using Wazuh as an analyst---searching
events, interpreting Windows Event IDs, identifying affected accounts,
pivoting on SIDs, and analyzing authentication activity to reconstruct
what occurred on monitored endpoints.
