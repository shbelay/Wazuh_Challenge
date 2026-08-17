# Wazuh Challenge - Part 4: Build Your SOC Dashboard

## Overview

In this module, I built a custom **SOC dashboard in Wazuh** to turn
collected security telemetry into visualizations that can be reviewed
quickly during monitoring and investigation.

Using the Wazuh dashboard visualization tools, I created panels for:

-   Failed Windows logons
-   Windows account changes
-   Linux authentication activity

The completed dashboard combines Windows and Linux security telemetry
into a single view.

------------------------------------------------------------------------

## Objectives

-   Create a custom Wazuh SOC dashboard
-   Build a metric visualization for failed Windows logons
-   Identify the Windows Event ID associated with failed logons
-   Build a visualization for Windows account-management activity
-   Monitor multiple Windows account-management Event IDs
-   Build a Linux authentication activity table
-   Filter telemetry by Wazuh agent
-   Break Linux authentication events down by useful investigation
    fields
-   Combine the visualizations into one SOC dashboard

------------------------------------------------------------------------

# 1. Create a New Dashboard

From the Wazuh interface, I navigated to:

``` text
Explore
└── Dashboards
```
<img width="317" height="473" alt="image" src="https://github.com/user-attachments/assets/f4f0aef2-976e-44fc-8738-ed4e4e7b4f4d" />

Because this was a new dashboard, I selected:

``` text
Create new dashboard
```
<img width="642" height="375" alt="image" src="https://github.com/user-attachments/assets/b7751b1d-cb18-49f1-8154-8f93838107dc" />

I then selected:

``` text
Create new
```

to begin adding the first visualization.

<img width="412" height="267" alt="image" src="https://github.com/user-attachments/assets/783db4a6-fc6f-4f44-90c3-6327a757a8fe" />

------------------------------------------------------------------------

# 2. Create a Failed Windows Logon Metric

For the first panel, I selected the **Metric** visualization type.

A metric visualization provides a simple count that can quickly show how
many events match a security condition.

<img width="847" height="712" alt="image" src="https://github.com/user-attachments/assets/4c7ccf57-9677-42d4-a5b1-125b63d2c1c1" />

## Select the Data Source

I selected the Wazuh archive data source:

``` text
wazuh-arc*
```

This allowed the visualization to query the archived endpoint telemetry
collected by Wazuh.

<img width="758" height="372" alt="image" src="https://github.com/user-attachments/assets/02a53e28-0745-4cbc-aad6-f4a9a4e8bc08" />

------------------------------------------------------------------------

# 3. Filter for Failed Windows Logons

I researched the Windows Security Event ID associated with failed logon
attempts.

The relevant Event ID is:

``` text
4625 – An account failed to log on
```

<img width="718" height="377" alt="image" src="https://github.com/user-attachments/assets/7f6e7e46-df06-4088-8bd2-a6cc736b2964" />

I then filtered the visualization using the Wazuh Windows Event ID
field:

``` text
data.win.system.eventID: 4625
```

This limited the metric to Windows failed-logon events.

<img width="323" height="86" alt="image" src="https://github.com/user-attachments/assets/6b7b66b0-c5fd-48fe-834e-f5b873d68e23" />

------------------------------------------------------------------------

# 4. Save the Failed Logon Visualization

After confirming the metric returned results, I selected **Save**.

I named the visualization:

``` text
Failed Windows Logon
```

I enabled the option to add the visualization to the dashboard after
saving and selected:

``` text
Save and return
```
<img width="448" height="433" alt="image" src="https://github.com/user-attachments/assets/9945e08a-4df7-477d-9e9a-29ec4745c40a" />

The new dashboard now contained a metric panel showing the number of
failed Windows logons within the selected time range.

<img width="958" height="782" alt="image" src="https://github.com/user-attachments/assets/9723e641-1944-4705-9262-b3373c0fda3c" />

------------------------------------------------------------------------

# 5. Save the SOC Dashboard

After creating the first visualization, I saved the dashboard itself.

I named it:

``` text
BelaySOC-Dashboard
```
<img width="421" height="477" alt="image" src="https://github.com/user-attachments/assets/547ce2ab-7f1a-4747-8a14-3140e01d3f45" />

<img width="401" height="78" alt="image" src="https://github.com/user-attachments/assets/a3ede958-62a2-4a71-b048-461fb3d1cafa" />

The dashboard could now be reopened and expanded with additional SOC
monitoring panels.

------------------------------------------------------------------------

# 6. Add a Windows Account Changes Visualization

I placed the dashboard into **Edit** mode and selected:

<img width="541" height="109" alt="image" src="https://github.com/user-attachments/assets/96c1685a-0353-4444-830b-daef941fc07d" />


``` text
Add
└── Create new
    └── Visualization
```

<img width="975" height="360" alt="image" src="https://github.com/user-attachments/assets/873ebd32-857f-447b-abcc-db9c5813d276" />

For this panel, I selected a **Line** visualization.

<img width="841" height="722" alt="image" src="https://github.com/user-attachments/assets/1db19d98-59ff-4d90-8dc4-551ec08c29b8" />

I again selected:

``` text
wazuh-arc*
```

as the source.

<img width="766" height="332" alt="image" src="https://github.com/user-attachments/assets/6e4bea05-a4a3-4db2-9047-c9733c8bb4e0" />

------------------------------------------------------------------------

# 7. Monitor Windows Account-Management Events

To monitor account-related activity, I created a query containing
several Windows Security Event IDs:

``` text
data.win.system.eventID: ("4720" OR "4722" OR "4723" OR "4724" OR "4725" OR "4726" OR "4732" OR "4733" OR "4738")
```

<img width="877" height="88" alt="image" src="https://github.com/user-attachments/assets/556f40fe-95be-4c4a-a9bd-662a8244b1f0" />

<img width="456" height="70" alt="image" src="https://github.com/user-attachments/assets/4b64cab6-ccf7-4eac-9bb3-b546d0ac31bf" />

These Event IDs represent several important account-management and
group-membership activities.

  Event ID   Security Activity
  ---------- ----------------------------------------------------------
  **4720**   A user account was created
  **4722**   A user account was enabled
  **4723**   An attempt was made to change an account's password
  **4724**   An attempt was made to reset an account's password
  **4725**   A user account was disabled
  **4726**   A user account was deleted
  **4732**   A member was added to a security-enabled local group
  **4733**   A member was removed from a security-enabled local group
  **4738**   A user account was changed

This query created a reusable view of Windows account-management
activity instead of requiring each Event ID to be searched separately.

------------------------------------------------------------------------

# 8. Save the Windows Account Changes Panel

I saved the visualization with the title:

``` text
Windows Account Changes
```
<img width="448" height="403" alt="image" src="https://github.com/user-attachments/assets/5d691df0-6adc-4aae-9a9d-0553ad7bdc56" />

I then returned to the dashboard.

At this point, the SOC dashboard contained panels for:

``` text
Failed Windows Logon
Windows Account Changes
```

------------------------------------------------------------------------

# 9. Create a Linux Authentication Activity Table

Next, I created another visualization for the Ubuntu endpoint.

From the dashboard, I selected:

``` text
Edit
└── Add
    └── Create new
        └── Visualization
```

<img width="242" height="122" alt="image" src="https://github.com/user-attachments/assets/cb3358b6-09c5-4f9e-acbc-062e1c8f3655" />

<img width="978" height="338" alt="image" src="https://github.com/user-attachments/assets/5a15142b-a5f3-4911-a583-e6eea974a67a" />

For this panel, I selected the **Data Table** visualization type.

<img width="885" height="733" alt="image" src="https://github.com/user-attachments/assets/2564f797-1275-4927-aff9-7d4a37851f43" />

I again used:

``` text
wazuh-arc*
```

as the data source.

<img width="758" height="372" alt="image" src="https://github.com/user-attachments/assets/9aff64ac-a5cc-4c7b-8b82-9adaccfdd856" />

------------------------------------------------------------------------

# 10. Filter for the Ubuntu Agent

I added a filter using:

``` text
Field: agent.name
Operator: is
Value: Ubuntu-Agent
```
<img width="660" height="358" alt="image" src="https://github.com/user-attachments/assets/9070cee4-ad89-4c4b-b384-40689163afcf" />

Conceptually, the filter was:

``` text
Failed password
```

This restricted the table to telemetry generated by the Ubuntu endpoint.

The visualization returned events associated with `Failed password`.

<img width="377" height="233" alt="image" src="https://github.com/user-attachments/assets/daeb9ac3-53bb-400a-ae68-e220ad0bf3d2" />

------------------------------------------------------------------------

# 11. Split the Table by Agent Name

Under **Buckets**, I selected:

``` text
Add
└── Split rows
```

<img width="577" height="242" alt="image" src="https://github.com/user-attachments/assets/0fa0f9ba-fab1-4b67-9675-83c1a9741a4c" />


I configured the aggregation as:

``` text
Aggregation: Terms
Field: agent.name
Order by: Metric Count
Order: Descending
```

After selecting **Update**, the table displayed the Ubuntu agent and its
corresponding event count.

<img width="576" height="571" alt="image" src="https://github.com/user-attachments/assets/95bff5b3-acfb-45bd-8fa4-b1a57bb61266" />

------------------------------------------------------------------------

# 12. Add Timestamp Information

I added another row split using:

``` text
Aggregation: Terms
Field: timestamp
Order by: Metric Count
Order: Descending
```

<img width="580" height="567" alt="image" src="https://github.com/user-attachments/assets/68313457-9d88-4fb6-abba-de37a6ad302b" />


This added timestamps to the table so I could see when
authentication-related events occurred.

The table now contained:

``` text
agent.name
timestamp
Count
```

<img width="1013" height="207" alt="image" src="https://github.com/user-attachments/assets/3b4e5856-fd89-432b-9871-a05cd6dead2e" />

------------------------------------------------------------------------

# 13. Add the Authentication User

I added another **Terms** aggregation using:

``` text
Field: data.dstuser
```

This field provided the destination/authentication username associated
with the event.

The table now provided additional user context for the Linux
authentication telemetry.

------------------------------------------------------------------------

# 14. Add the Source User

I added another Terms aggregation using:

``` text
Field: data.srcuser
```
<img width="572" height="563" alt="image" src="https://github.com/user-attachments/assets/95ff04fa-addc-4d0c-9e32-1b9bab64c31f" />

This provided another user-related field from the parsed authentication
event.

In the captured telemetry, some events showed:

``` text
Missing
```

for this value, demonstrating that not every parsed field is populated
for every event.

------------------------------------------------------------------------

# 15. Add the Source IP Address

Finally, I added another Terms aggregation using:

``` text
Field: data.srcip
```

This added the originating IP address to the table.

The final Linux authentication table contained fields similar to:

  Field            Purpose
  ---------------- ---------------------------------------
  `agent.name`     Endpoint that generated the telemetry
  `timestamp`      Time the event occurred
  `data.dstuser`   Authentication/destination user
  `data.srcuser`   Source user when available
  `data.srcip`     Source IP address
  `Count`          Number of matching events

<img width="1312" height="251" alt="image" src="https://github.com/user-attachments/assets/29d810c8-c511-4cf9-b3b8-8f4b924f53ff" />

This created a much more useful analyst view than a simple event count
because I could immediately see **which system, when, which user, and
which source IP** were associated with the authentication activity.

------------------------------------------------------------------------

# 16. Save the Linux Authentication Visualization

After completing the table, I selected **Save**.

<img width="467" height="67" alt="image" src="https://github.com/user-attachments/assets/64f40d14-faf4-4f96-b3f5-d7ffcd3147bf" />

I named the visualization:

``` text
Linux Authentication Activity
```

I enabled:

``` text
Add to Dashboards after saving
```

and selected:

``` text
Save and return
```

<img width="450" height="432" alt="image" src="https://github.com/user-attachments/assets/1515ccc0-8269-4c28-bac2-21abb5e2a654" />

------------------------------------------------------------------------

# 17. Review the Completed SOC Dashboard

The completed dashboard contained three security-focused panels:

``` text
BelaySOC-Dashboard
│
├── Failed Windows Logon
│   └── Windows Event ID 4625
│
├── Windows Account Changes
│   ├── 4720 – Account Created
│   ├── 4722 – Account Enabled
│   ├── 4723 – Password Change Attempt
│   ├── 4724 – Password Reset Attempt
│   ├── 4725 – Account Disabled
│   ├── 4726 – Account Deleted
│   ├── 4732 – Member Added to Local Group
│   ├── 4733 – Member Removed from Local Group
│   └── 4738 – Account Changed
│
└── Linux Authentication Activity
    ├── Agent
    ├── Timestamp
    ├── Destination User
    ├── Source User
    └── Source IP
```

The screenshots show the dashboard successfully displaying the **Failed
Windows Logon** metric and **Linux Authentication Activity** table. The
**Windows Account Changes** panel was also present but showed no results
for the dashboard's selected time window at the time of the final
screenshot.

<img width="1898" height="731" alt="image" src="https://github.com/user-attachments/assets/6ab77e6d-c5c3-47b7-8c89-550d2c92d79d" />

------------------------------------------------------------------------

# SOC Value of the Dashboard

A dashboard provides an analyst with a high-level view of security
activity without requiring individual searches for every monitoring use
case.

The panels created in this module can help surface:

### Authentication Activity

``` text
Windows Event ID 4625
        ↓
Failed Windows Logons
        ↓
Potential brute-force or unauthorized access investigation
```

### Account Management Activity

``` text
Windows Account Event IDs
        ↓
Account Creation / Modification / Deletion
        ↓
Privileged Group Changes
        ↓
Potential persistence or privilege escalation investigation
```

### Linux Authentication Activity

``` text
Ubuntu Authentication Logs
        ↓
Username + Timestamp + Source IP
        ↓
Authentication Investigation
```

------------------------------------------------------------------------

# Key Takeaways

In this module, I practiced:

-   Creating Wazuh dashboards
-   Building custom security visualizations
-   Using metric visualizations
-   Using line visualizations
-   Building data tables
-   Filtering on Windows Event IDs
-   Filtering telemetry by endpoint
-   Using Terms aggregations
-   Splitting table rows by multiple fields
-   Monitoring Windows authentication activity
-   Monitoring Windows account-management activity
-   Monitoring Linux authentication activity
-   Presenting security telemetry in an analyst-friendly format

A key takeaway from this module was that collecting telemetry is only
one part of a SIEM deployment. The data must also be organized into
useful views that allow an analyst to quickly identify activity
requiring further investigation.

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
├── Part 3 – Generate + Read Telemetry
│   ├── Windows Reconnaissance
│   ├── Account Management Events
│   ├── SID Correlation
│   └── Linux SSH Authentication Analysis
│
└── Part 4 – Build Your SOC Dashboard
    ├── Failed Windows Logon
    ├── Windows Account Changes
    └── Linux Authentication Activity
```

------------------------------------------------------------------------

# Outcome

I successfully created a custom Wazuh SOC dashboard that brings Windows
and Linux security telemetry into a centralized monitoring view.

This module moved the project from **searching individual events to
operationalizing the telemetry for continuous SOC monitoring**. By
creating visualizations for failed authentication, Windows account
changes, and Linux authentication activity, I built a dashboard that can
help an analyst identify suspicious activity and quickly determine where
deeper investigation is needed.
