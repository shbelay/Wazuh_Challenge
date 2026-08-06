# Wazuh Challenge - Part 1: Deploy the Server

For part 1 of this guide, I walk through deploying a **Wazuh server** on an Ubuntu 24.04 machine, enabling archive logging, configuring Filebeat, and creating an index pattern in OpenSearch so raw event logs are searchable.

---

# Prerequisites

- Ubuntu 24.04 Server
- Internet connectivity
- SSH access to the server
- Web browser

---

# Step 1 - Install Wazuh

Visit the Wazuh website:

https://wazuh.com/

<img width="1461" height="720" alt="image" src="https://github.com/user-attachments/assets/23a91d67-4d02-4f2a-b8ff-be61a983c48e" />

Select:

**Install Wazuh → Quick Start**

<img width="1433" height="602" alt="image" src="https://github.com/user-attachments/assets/f9036c1a-a2b0-44db-8257-417aeb7aab25" />

Copy the installation command.

Example:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

<img width="1161" height="195" alt="image" src="https://github.com/user-attachments/assets/3eef2426-28f0-4039-aac5-e487523002f8" />

Run the command from your Ubuntu server.

Installation typically completes within **5–10 minutes**.

<img width="1498" height="152" alt="image" src="https://github.com/user-attachments/assets/7e2027ab-b16a-4d05-90da-287e799d0eaf" />


At the end of the installation you will receive something similar to:

```

INFO: You can access the web interface
https://<wazuh-dashboard-ip>:443

User: admin
Password: <generated password>

```
<img width="1152" height="153" alt="image" src="https://github.com/user-attachments/assets/499dba91-8ef0-4962-9dbf-f1bfe036520a" />
---

# Step 2 - Log into the Dashboard

Open your browser and navigate to:

```

https://<wazuh-dashboard-ip>

```

Login using the credentials generated during installation.

At this point the Wazuh dashboard should be accessible and collecting alerts from the server.

<img width="1897" height="1036" alt="image" src="https://github.com/user-attachments/assets/19022c13-b898-4653-bf3e-ce188fdeb190" />

---

# Understanding Wazuh Data

<img width="1912" height="915" alt="image" src="https://github.com/user-attachments/assets/a341e1dd-5384-4af9-9799-ed1c4c94c436" />

Wazuh stores two types of data.

## Alerts

Alerts are events that match detection rules.

- Indexed automatically
- Visible in the dashboard

## Archives

Archives contain the raw logs received from agents before rule processing.

By default:

- Archive logs are **not indexed**
- Successful logons and many benign events are not searchable

Enabling archives provides full visibility into all collected events.

---

# Step 3 - Verify Archive Directory

Navigate to the archive directory.

```bash
cd /var/ossec
```

List its contents.

```bash
ls -la /var/ossec
```

<img width="577" height="521" alt="image" src="https://github.com/user-attachments/assets/538daa12-cff1-4c3e-a109-dba4d2d6bb5a" />

---

# Step 4 - Enable Archive Logging

Navigate to the Wazuh configuration directory.

```bash
cd /var/ossec/etc
```
<img width="946" height="100" alt="image" src="https://github.com/user-attachments/assets/aa8a0925-d6c5-4275-b6d2-6dd9af46cafb" />

Open the configuration file.

```bash
nano ossec.conf
```

Locate:

```xml
<logall>no</logall>
```
<img width="667" height="457" alt="image" src="https://github.com/user-attachments/assets/4d63b68c-9b86-4e57-8c24-d5b4b5fe3f05" />

Change it to:

```xml
<logall>yes</logall>
```

Save the file.

<img width="671" height="465" alt="image" src="https://github.com/user-attachments/assets/aacf5969-924a-4001-8bc1-05dde333bd2e" />

---

# Step 5 - Restart Wazuh Manager

Restart the service to apply the configuration.

```bash
systemctl restart wazuh-manager.service
```
<img width="760" height="85" alt="image" src="https://github.com/user-attachments/assets/6c40395c-3494-4f82-97b7-936e13314644" />

---

# Step 6 - Configure Filebeat

Navigate to the Filebeat directory.

```bash
cd /etc/filebeat
```
<img width="635" height="285" alt="image" src="https://github.com/user-attachments/assets/01cb5926-2692-4653-8068-98c6e3f37113" />

Open the configuration.

```bash
nano filebeat.yml
```

Locate:

```yaml
archives:
  enabled: false
```
<img width="308" height="167" alt="image" src="https://github.com/user-attachments/assets/91d52cde-479f-4624-ba03-7cf6d6ef9a07" />

Change it to:

```yaml
archives:
  enabled: true
```
<img width="582" height="502" alt="image" src="https://github.com/user-attachments/assets/f04ff81b-792b-4f1a-b864-55c08b505620" />

This instructs Filebeat to forward archive logs into OpenSearch.

Save the configuration.

---

# Step 7 - Restart Filebeat

Restart Filebeat.

```bash
systemctl restart filebeat.service
```
<img width="723" height="125" alt="image" src="https://github.com/user-attachments/assets/77d2bb7b-9a4a-497b-a825-754902a0a0fa" />

---

# Step 8 - Create an Archive Index Pattern

Within the Wazuh dashboard navigate to:

```

Dashboard Management
└── Index Patterns

```
<img width="330" height="850" alt="image" src="https://github.com/user-attachments/assets/6de0e5b0-3c67-4964-8c55-c414bf8de209" />

<img width="265" height="167" alt="image" src="https://github.com/user-attachments/assets/5f8f6d51-9c6d-4305-b66e-da4e830b00c7" />

Click:

**Create Index Pattern**

<img width="1268" height="466" alt="image" src="https://github.com/user-attachments/assets/98b88078-944e-40d4-ae39-1948817eeb42" />

Search for:

```

wazuh-archives

```

Select the archive index.

Click:

**Next Step**

<img width="1241" height="555" alt="image" src="https://github.com/user-attachments/assets/918a4c25-0270-4e54-a6cc-3ac982254fa4" />

For the time field select:

```

timestamp

```

Finally click:

**Create Index Pattern**

<img width="1238" height="557" alt="image" src="https://github.com/user-attachments/assets/b9639ade-95b3-4f3c-9302-3d4598e00c73" />

---

# Step 9 - Verify Archive Data

Navigate to:

```

Explore
└── Discover

```
<img width="307" height="315" alt="image" src="https://github.com/user-attachments/assets/66ce837a-311e-4a20-a589-8c4de43fe930" />

You should now see:

```

wazuh-archives

```

available as a searchable index.

Archive events are now indexed and searchable inside OpenSearch.

<img width="410" height="228" alt="image" src="https://github.com/user-attachments/assets/489e4ecb-122f-498f-b0e7-9a7e0b37ebaa" />

---

# Summary

In this lab you:

- Installed Wazuh on Ubuntu 24.04
- Accessed the Wazuh Dashboard
- Enabled archive logging
- Modified the `ossec.conf` configuration
- Restarted the Wazuh Manager
- Configured Filebeat to index archive logs
- Restarted Filebeat
- Created a `wazuh-archives` index pattern
- Verified archive logs are searchable in OpenSearch

---

# Outcome

The Wazuh server is successfully deployed and configured to collect both **alerts** and **raw archive events**, providing complete visibility into endpoint activity for future investigations.
