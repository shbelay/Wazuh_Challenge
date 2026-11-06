# Wazuh Challenge -- Setting Up SOAR with Tines

## Overview

In this module, I integrated **Wazuh** with **Tines** to build a
Security Orchestration, Automation, and Response (**SOAR**) workflow.

The workflow receives a Wazuh alert through a Tines webhook, enriches
the alert with an AI Agent and external intelligence tools, sends the
analysis to Slack, extracts a recommendation and IOC from the AI output,
and uses conditional logic to determine whether the SOC analyst should
be prompted to block the IOC.

For the response portion, I configured Tines to communicate back to the
Wazuh API and invoke Wazuh's `firewall-drop` Active Response when the
analyst approves the block.

------------------------------------------------------------------------

## Objectives

-   Create a Tines tenant and Story
-   Create a webhook to receive Wazuh alerts
-   Configure Wazuh to send a custom rule alert to Tines
-   Validate Wazuh-to-Tines alert delivery
-   Add an AI Agent for alert enrichment
-   Add AbuseIPDB and VirusTotal enrichment tools
-   Send enriched findings to Slack
-   Extract AI-generated IOC and response recommendations
-   Add conditional SOAR decision logic
-   Require analyst approval before blocking an IOC
-   Configure Tines to communicate with the Wazuh API
-   Trigger Wazuh Active Response from the SOAR workflow
-   Validate the end-to-end response process

------------------------------------------------------------------------

# 1. Create a Tines Tenant

I logged in to Tines, opened my profile menu, and selected:

``` text
Switch Tenant
```

<img width="251" height="328" alt="image" src="https://github.com/user-attachments/assets/69e16529-b49c-4f90-93e9-b7e90edde692" />

I then selected:

``` text
New
└── Create new tenant
```

<img width="361" height="162" alt="image" src="https://github.com/user-attachments/assets/90b8c4fc-419d-4bd0-9463-80335655b80a" />

From the tenant type menu, I selected:

``` text
Tines Stories
```

<img width="405" height="256" alt="image" src="https://github.com/user-attachments/assets/b246e29a-350f-4a30-8388-e2063ecaf0a2" />

and clicked:

``` text
Create tenant
```

<img width="361" height="340" alt="image" src="https://github.com/user-attachments/assets/a65d7218-f36d-48e9-b0e4-395f84df6f3a" />

This provided a workspace where I could build the SOAR workflow.

------------------------------------------------------------------------

# 2. Create the Wazuh Webhook

Inside the Tines Story, I selected a:

``` text
Webhook
```

and dragged it onto the Story canvas.

I named the webhook for the Wazuh integration and copied the generated:

``` text
Webhook URL
```

<img width="1911" height="477" alt="image" src="https://github.com/user-attachments/assets/c29de4b9-dc22-4e21-853a-4ade03aac494" />

This URL would become the destination to which Wazuh sends matching
security alerts.

Conceptually:

``` text
Wazuh Alert
     │
     ▼
Tines Webhook
     │
     ▼
SOAR Workflow
```

------------------------------------------------------------------------

# 3. Configure Wazuh to Send Alerts to Tines

I connected to the Wazuh Manager over SSH and edited:

``` text
/var/ossec/etc/ossec.conf
```

<img width="537" height="77" alt="image" src="https://github.com/user-attachments/assets/5c848c48-765d-44e5-826f-6a3770f16a8a" />

Under the global configuration, I added a custom integration similar to:

``` xml
<integration>
  <name>custom-tines</name>
  <hook_url><TINES_WEBHOOK_URL></hook_url>
  <alert_format>json</alert_format>
  <rule_id>100101</rule_id>
</integration>
```

<img width="978" height="592" alt="image" src="https://github.com/user-attachments/assets/f7af46d7-84ed-4cef-9d9f-bbb8295100bd" />

The important values were:

  Setting          Purpose
  ---------------- --------------------------------------------------------
  `custom-tines`   Identifies the custom integration
  `hook_url`       Tines webhook receiving the Wazuh event
  `alert_format`   Sends the alert in JSON format
  `rule_id`        Limits forwarding to the selected Wazuh detection rule

The `rule_id` used in the lab was:

``` text
100101
```

This corresponded to the custom Wazuh rule previously created for:

``` text
Multiple SSH login failures observed from the same source IP
```

------------------------------------------------------------------------

# 4. Restart the Wazuh Manager

After updating `ossec.conf`, I restarted the Wazuh Manager service so
the integration configuration would take effect.

<img width="661" height="57" alt="image" src="https://github.com/user-attachments/assets/da59f2d1-1b41-40ee-afeb-eef26ea8b1ac" />

------------------------------------------------------------------------

# 5. Generate an Alert

From my Windows test machine, I intentionally generated failed SSH
authentication attempts against the Wazuh Manager.

The goal was to trigger Wazuh rule:

``` text
100101
```

When the rule fired, Wazuh sent the alert to the Tines webhook.

<img width="802" height="370" alt="image" src="https://github.com/user-attachments/assets/67e45d15-1aec-4723-9a9e-f36825c808ba" />

------------------------------------------------------------------------

# 6. Validate the Event in Tines

I returned to Tines and reviewed the webhook event.

The event contained the Wazuh alert in JSON format, including the custom
rule information.

This confirmed the first portion of the SOAR pipeline:

``` text
Failed SSH Attempts
        │
        ▼
Wazuh Detection
        │
        ▼
Rule 100101
        │
        ▼
Custom Tines Integration
        │
        ▼
Tines Webhook
```

<img width="1342" height="760" alt="image" src="https://github.com/user-attachments/assets/08bc39a0-2045-4dc7-8775-82768bd30a77" />

------------------------------------------------------------------------

# 7. Add an AI Agent

To enrich and analyze the incoming alert, I added an:

``` text
AI Agent
```

and selected:

``` text
Task
```

<img width="427" height="232" alt="image" src="https://github.com/user-attachments/assets/bf2e67a7-b901-47ee-917f-ac956265c3a6" />

I dragged the AI Agent onto the Story canvas and connected:

``` text
Wazuh Webhook
      │
      ▼
AI Agent
```

The AI Agent would receive the Wazuh event and use the configured tools
and instructions to analyze it.

<img width="302" height="267" alt="image" src="https://github.com/user-attachments/assets/1015661f-9bb0-4037-b679-b9e091300847" />

------------------------------------------------------------------------

# 8. Add AbuseIPDB

From the AI Agent configuration, I selected:

``` text
Add tool
```

I searched for:

``` text
AbuseIPDB
```

<img width="777" height="512" alt="image" src="https://github.com/user-attachments/assets/63e2a159-ec3b-4b83-956c-3796ef2962fe" />

and selected:

``` text
Search for an IP address
```

<img width="385" height="542" alt="image" src="https://github.com/user-attachments/assets/83360fa7-9d2c-4cec-a106-2fd592a11428" />

This gave the AI Agent a way to enrich an IP address associated with the
Wazuh alert.

------------------------------------------------------------------------

# 9. Add VirusTotal

I selected **Add tool** again and searched for:

``` text
VirusTotal
```

<img width="711" height="482" alt="image" src="https://github.com/user-attachments/assets/3b29e772-f909-4b35-8bfa-aa198edda8fa" />

I selected:

``` text
Search for files, URLs, domains, IPs and comments
```

<img width="308" height="476" alt="image" src="https://github.com/user-attachments/assets/fa54275f-ddb2-4843-9662-45050e52ea8b" />

This added another enrichment source to the AI Agent.

------------------------------------------------------------------------

# 10. Add Slack

I added another tool and searched for:

``` text
Slack
```

<img width="341" height="477" alt="image" src="https://github.com/user-attachments/assets/b061cf71-eed2-4136-9634-c1d639155656" />

I selected:

``` text
Send a message
```

<img width="307" height="480" alt="image" src="https://github.com/user-attachments/assets/338769d1-b151-4459-bcce-248e6b87c3bc" />

This allowed the workflow to send the AI-enriched security finding to a
Slack channel for analyst review.

------------------------------------------------------------------------

# 11. Configure Tool Connections

After adding AbuseIPDB, VirusTotal, and Slack, I configured their
connections.

For each tool, I used the **Credentials** section and selected:

``` text
Connect
└── New connection
```

<img width="707" height="527" alt="image" src="https://github.com/user-attachments/assets/8fd2fd84-36ee-4d8d-b3af-d6dd97e5e3d7" />

I followed the connection steps for each service.

Once configured successfully, the applications displayed green
connection indicators.

The AI Agent now had access to:

``` text
AI Agent
│
├── AbuseIPDB
├── VirusTotal
└── Slack
```

<img width="361" height="295" alt="image" src="https://github.com/user-attachments/assets/1e79f5fb-0d1c-40ed-aad7-49c59abe5f3f" />

------------------------------------------------------------------------

# 12. Configure the AI Agent Prompt

I opened the AI Agent and populated the:

``` text
System Instructions
```

and:

``` text
Prompt
```

fields.

The prompt was designed to analyze the Wazuh security event, enrich
relevant indicators, and produce a structured recommendation that could
later be parsed by the workflow.

<img width="352" height="656" alt="image" src="https://github.com/user-attachments/assets/2692e8f8-73b1-43e7-b30d-1b78535f33ce" />

To pass the complete Wazuh alert into the prompt, I hovered over the
Prompt field, selected the **plus (+)** button, and chose:

``` text
Value
```

<img width="357" height="364" alt="image" src="https://github.com/user-attachments/assets/2697cbec-630d-470e-8cab-db535c03828d" />

<img width="177" height="102" alt="image" src="https://github.com/user-attachments/assets/150bcda1-7596-47ff-aa4e-6fb5bdd98977" />

I then selected:

``` text
Wazuh-Webhook
└── Body
    └── All-fields
```

This inserted the webhook event data into the AI Agent prompt.

<img width="340" height="380" alt="image" src="https://github.com/user-attachments/assets/5105eff7-75a2-4fbe-a8cf-3f4fe8bd7ead" />

------------------------------------------------------------------------

# 13. Test the AI Enrichment Workflow

To test the workflow without generating another alert from scratch, I
selected the previously received webhook event and clicked:

``` text
Re-emit
```

<img width="1036" height="807" alt="image" src="https://github.com/user-attachments/assets/6bb632e6-9752-4aa5-988a-b7d5c8c7d50d" />

Tines replayed the event through the workflow.

I waited for the AI Agent task to complete and then checked the
configured Slack channel.

<img width="477" height="352" alt="image" src="https://github.com/user-attachments/assets/ce694d2a-7cc0-4f5f-8f2a-61a3df530fb8" />

The enriched security finding appeared in Slack.

<img width="1907" height="857" alt="image" src="https://github.com/user-attachments/assets/5e5a9ef3-9943-4764-8ef8-9928bf6612de" />

This validated:

``` text
Wazuh
  │
  ▼
Tines Webhook
  │
  ▼
AI Agent
  │
  ├── AbuseIPDB
  ├── VirusTotal
  │
  ▼
Slack Finding
```

------------------------------------------------------------------------

# 14. Add Active Response Logic

The next goal was to extend the workflow beyond enrichment and
notification.

I wanted the AI Agent to recommend whether an IOC should be blocked,
while still requiring a **SOC analyst decision** before executing the
response.

The intended workflow was:

``` text
AI Analysis
     │
     ▼
Block Recommended?
     │
     ├── No ──► Do Not Block
     │
     └── Yes
          │
          ▼
    Ask SOC Analyst
          │
          ▼
    Analyst Approves?
          │
          ├── No ──► Stop
          │
          └── Yes
                 │
                 ▼
          Wazuh Active Response
```

------------------------------------------------------------------------

# 15. Add Event Transform

I added an:

``` text
Event Transform
```

and connected it to the AI Agent.

<img width="399" height="472" alt="image" src="https://github.com/user-attachments/assets/4dbd0d20-af21-4d04-a2e7-61cb7af44d4b" />
<img width="492" height="491" alt="image" src="https://github.com/user-attachments/assets/d38d34a9-8db4-4ede-8d62-15ad686a23f0" />


The Event Transform was used to extract specific values from the AI
Agent's output.

I configured the path:

``` text
ai_agent.output
```

<img width="927" height="382" alt="image" src="https://github.com/user-attachments/assets/ba952a15-e3c9-4ac9-8b5c-9c5169c614db" />
<img width="890" height="402" alt="image" src="https://github.com/user-attachments/assets/38c74320-2f03-42dc-833a-7cda38e7fbd8" />

------------------------------------------------------------------------

# 16. Extract the AI Recommendation

I configured a Regex extraction for:

``` text
"recommend_block": (\w+)
```

and stored the result as:

``` text
recommend_block
```

<img width="367" height="211" alt="image" src="https://github.com/user-attachments/assets/57079c07-365d-4656-ab02-5c1fadefd7b9" />

I then created two additional extractions.

### IOC Type

``` text
"ioc_type": "(\w+)"
```

Stored as:

``` text
ioc_type
```

### IOC Value

``` text
"ioc_value": "(.+)"
```

Stored as:

``` text
ioc_value
```

<img width="367" height="377" alt="image" src="https://github.com/user-attachments/assets/239f15c0-b5f9-4b1d-8911-5bcec19f2902" />

The Event Transform therefore converted the AI output into values that
could be used by later workflow logic:

``` text
AI Agent Output
      │
      ▼
Event Transform
      │
      ├── recommend_block
      ├── ioc_type
      └── ioc_value
```

------------------------------------------------------------------------

# 17. Add Conditional Logic

I added two **Conditions** to the Story and connected both to the Event
Transform.

I named them:

``` text
Block is recommended
```

<img width="320" height="192" alt="image" src="https://github.com/user-attachments/assets/1d4f026e-4c59-4450-ac42-065f8661456b" />

<img width="520" height="332" alt="image" src="https://github.com/user-attachments/assets/80b3b63c-aa5f-4374-bb4b-000a0956c40c" />

and:

``` text
Don't Block
```

For **Block is recommended**, I configured the rule:

``` text
TEXT(event_transform.recommend_block) is equal to true
```

<img width="387" height="202" alt="image" src="https://github.com/user-attachments/assets/3c1bf588-6bd0-44e5-8624-c3085bf5f50c" />

For **Don't Block**, I configured:

``` text
TEXT(event_transform.recommend_block) is equal to false
```

<img width="367" height="182" alt="image" src="https://github.com/user-attachments/assets/8b478822-8deb-4e92-86b7-4d6742c30549" />

This created two workflow paths based on the AI Agent's recommendation.

------------------------------------------------------------------------

# 18. Create an Analyst Approval Page

If blocking was recommended, I did not want the automation to
immediately block the IOC.

Instead, I created a Tines:

``` text
Page
```

<img width="911" height="587" alt="image" src="https://github.com/user-attachments/assets/1cedf1c9-7dc8-4e80-8154-ee13030545ce" />

and connected:

``` text
Block is recommended
        │
        ▼
Page
```

<img width="336" height="445" alt="image" src="https://github.com/user-attachments/assets/2ce0d14b-69ed-4594-9f44-ea56434f2c20" />

I opened the Page editor and created a simple analyst approval
interface.

<img width="440" height="667" alt="image" src="https://github.com/user-attachments/assets/8b7cb797-e43b-4ca5-9e42-acfd50be6a4a" />

The page asked the analyst whether the IOC should be blocked.

I added:

-   A Boolean control
-   A button

This introduced a **human-in-the-loop** decision before the response
action.

<img width="580" height="317" alt="image" src="https://github.com/user-attachments/assets/4fbf9483-253a-4442-8050-fed60660ac68" />

------------------------------------------------------------------------

# 19. Add the Analyst Approval Condition

Back in the Story, I added another Condition named:

``` text
Yes to Block
```

<img width="937" height="451" alt="image" src="https://github.com/user-attachments/assets/e4779302-9d1e-40b6-af0a-558ac702f598" />

<img width="1420" height="677" alt="image" src="https://github.com/user-attachments/assets/0fcdb53d-e862-4809-a676-5cc1d9ba5890" />

I connected the analyst approval Page to this Condition.

The rule checked whether the analyst selected the option to block the
IP:

``` text
page.body..do_you_want_to_block_the_ip is equal to true
```

<img width="425" height="247" alt="image" src="https://github.com/user-attachments/assets/639108b7-a232-4457-bac8-ba524c34f3bd" />

The response would therefore only continue when both conditions were
met:

``` text
AI recommends blocking
          +
SOC analyst approves blocking
          │
          ▼
Execute response
```

------------------------------------------------------------------------

# 20. Prepare Tines-to-Wazuh API Communication

The next step was to allow Tines to communicate back to the Wazuh
Manager.

<img width="377" height="289" alt="image" src="https://github.com/user-attachments/assets/9f5b902d-88e6-4bc5-a27f-f2fa248cb6de" />

In Tines, I selected:

``` text
New
└── Manual creation
```

<img width="569" height="170" alt="image" src="https://github.com/user-attachments/assets/899e6cdd-bb20-4fc2-99e9-8dc2d6c1576c" />

<img width="347" height="217" alt="image" src="https://github.com/user-attachments/assets/b7f0f62d-cc2e-404c-b394-3d3e2832f40d" />

I created an:

``` text
HTTP Request
```

credential.

<img width="867" height="777" alt="image" src="https://github.com/user-attachments/assets/2f862fd9-7781-44be-82a8-896e8809ccf6" />

------------------------------------------------------------------------

# 21. Provide Access to the On-Premises Wazuh API

My Wazuh Manager was hosted inside my lab environment rather than being
directly reachable from Tines.

To provide temporary lab connectivity, I used a tunnel service to expose
the required Wazuh API endpoint.

> The public tunnel URL is intentionally omitted from this write-up.
> Exposing management APIs publicly should only be done in a controlled
> lab with appropriate safeguards.

I then entered the Wazuh API credentials into the Tines HTTP Request
credential configuration.

<img width="657" height="575" alt="image" src="https://github.com/user-attachments/assets/d15b8f5c-3be2-4cdf-b8d4-933970fb0bff" />

<img width="672" height="580" alt="image" src="https://github.com/user-attachments/assets/5c4f0b97-fba2-49fc-961e-4ff2b164cd3a" />

------------------------------------------------------------------------

# 22. Add the Wazuh Action to the Story

I returned to the Story and searched for:

``` text
Wazuh
```

<img width="495" height="287" alt="image" src="https://github.com/user-attachments/assets/2332468a-f32b-4f4b-8123-3c219b08e5ad" />

I selected:

``` text
Run command in Wazuh
```

<img width="447" height="276" alt="image" src="https://github.com/user-attachments/assets/6f2f6a82-c71f-4ade-b35f-2280ba597de4" />

and dragged the action onto the Story.

I connected it to:

``` text
Yes to Block
```

<img width="235" height="275" alt="image" src="https://github.com/user-attachments/assets/710097f6-71fd-41b3-9f30-596795b9a7b4" />

The workflow now had a response path from analyst approval to Wazuh.

------------------------------------------------------------------------

# 23. Configure the Wazuh Active Response Request

In the Wazuh action configuration, I changed the URL to the temporary
tunnel URL that provided access to my lab Wazuh API.

<img width="896" height="430" alt="image" src="https://github.com/user-attachments/assets/797ba7f6-4ed0-4169-ba53-c767cbeeb8b3" />

Under the request payload, I configured the action to invoke Wazuh's:

``` text
!firewall-drop
```

Active Response command.

The payload structure used the IOC extracted earlier by the Event
Transform and supplied it to the Wazuh response as the source IP.

A sanitized representation of the payload is:

``` json
{
  "arguments": [
    "<<command_name>>"
  ],
  "command": "!firewall-drop",
  "alert": {
    "data": {
      "srcip": "<<TEXT(event_transform.ioc_value)>>"
    }
  }
}
```

<img width="357" height="416" alt="image" src="https://github.com/user-attachments/assets/437fbe7e-4e86-4569-8793-d3c806fe6c79" />

------------------------------------------------------------------------

# 24. End-to-End SOAR Workflow

The completed workflow can be represented as:

``` text
                    ┌───────────────────┐
                    │ Failed SSH Logins │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │       Wazuh       │
                    │    Rule 100101    │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   Tines Webhook   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │     AI Agent      │
                    ├───────────────────┤
                    │ AbuseIPDB         │
                    │ VirusTotal        │
                    │ Slack             │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Event Transform   │
                    ├───────────────────┤
                    │ recommend_block   │
                    │ ioc_type          │
                    │ ioc_value         │
                    └─────────┬─────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
       Block Recommended               Don't Block
                 │
                 ▼
          Analyst Approval
                 │
                 ▼
           Yes to Block
                 │
                 ▼
       Run Command in Wazuh
                 │
                 ▼
       Wazuh Active Response
                 │
                 ▼
           firewall-drop
```

------------------------------------------------------------------------

# 25. Validate Wazuh Active Response

To validate the completed workflow, I re-emitted a test event associated
with the lab authentication activity.

The event passed through the Tines workflow and generated the expected
response path.

The final lab screenshot confirmed that:

``` text
Wazuh Active Response worked via Tines AI
```

<img width="461" height="357" alt="image" src="https://github.com/user-attachments/assets/63b4a088-9a15-4538-a6ac-201dd9c14ad7" />

<img width="905" height="211" alt="image" src="https://github.com/user-attachments/assets/1f3ac794-1911-4a62-b06d-6dc80887c66f" />

This demonstrated an end-to-end SOAR process from **detection and
enrichment to analyst-approved response**.

------------------------------------------------------------------------

# SOAR Components Used

  -----------------------------------------------------------------------
  Component                           Role in the Workflow
  ----------------------------------- -----------------------------------
  **Wazuh**                           SIEM/detection source and Active
                                      Response platform

  **Tines Webhook**                   Receives the Wazuh JSON alert

  **AI Agent**                        Analyzes and enriches the security
                                      event

  **AbuseIPDB**                       IP reputation enrichment

  **VirusTotal**                      IOC reputation/context enrichment

  **Slack**                           Delivers the enriched finding to
                                      the SOC analyst

  **Event Transform**                 Extracts recommendation, IOC type,
                                      and IOC value

  **Conditions**                      Controls workflow branches

  **Tines Page**                      Provides human approval before
                                      blocking

  **Wazuh API Action**                Sends the approved response back to
                                      Wazuh

  **Wazuh Active Response**           Executes the firewall block
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Detection-to-Response Lifecycle

This module demonstrated a complete security operations lifecycle:

``` text
Detect
  │
  ▼
Collect
  │
  ▼
Forward
  │
  ▼
Enrich
  │
  ▼
Analyze
  │
  ▼
Notify
  │
  ▼
Recommend
  │
  ▼
Human Approval
  │
  ▼
Respond
  │
  ▼
Validate
```

------------------------------------------------------------------------

# Key Takeaways

In this module, I practiced:

-   Wazuh-to-SOAR integration
-   Tines Story development
-   Webhook-based alert forwarding
-   JSON alert handling
-   AI-assisted security enrichment
-   AbuseIPDB and VirusTotal enrichment
-   Slack security notifications
-   Event transformation and Regex extraction
-   Conditional SOAR logic
-   Human-in-the-loop response approval
-   API credential configuration
-   Wazuh API integration
-   Wazuh Active Response
-   Automated firewall blocking
-   End-to-end workflow testing

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
├── Part 7 – Setting Up a C2
│
└── Setting Up SOAR
    ├── Wazuh → Tines Webhook
    ├── AI Agent
    │   ├── AbuseIPDB
    │   ├── VirusTotal
    │   └── Slack
    ├── Event Transform
    │   ├── Block Recommendation
    │   ├── IOC Type
    │   └── IOC Value
    ├── Conditional Logic
    ├── Analyst Approval
    ├── Tines → Wazuh API
    └── Wazuh Active Response
        └── firewall-drop
```

------------------------------------------------------------------------

# Outcome

I successfully integrated Wazuh with Tines and built an end-to-end SOAR
workflow.

The workflow received a Wazuh alert, enriched the event using AI and
threat-intelligence sources, delivered the analysis to Slack, extracted
the AI's response recommendation and IOC, applied conditional logic,
requested analyst approval, and sent the approved action back to Wazuh.

The final test demonstrated that **Wazuh Active Response could be
triggered through the Tines workflow**, completing the progression from
SIEM detection to orchestrated and analyst-approved response.
