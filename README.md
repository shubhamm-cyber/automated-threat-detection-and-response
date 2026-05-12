# Automated Threat Intelligence Enrichment & Response Workflow

## Project Overview

Security analysts often spend significant time manually validating suspicious IP addresses, checking threat intelligence platforms, and deciding whether an alert requires immediate action or can be safely deprioritized.

To reduce this manual workload, I built an automated SOAR workflow using Microsoft Sentinel, Microsoft Logic Apps, VirusTotal, and Slack.

The workflow automatically enriches suspicious IP addresses with threat intelligence data, evaluates malicious reputation scores, and performs automated response actions whenever a security alert is triggered.

---

# Key Outcomes

✅ Automated threat intelligence enrichment for suspicious IP addresses
✅ Reduced manual investigation and repetitive analyst tasks
✅ Prioritized high-risk alerts using reputation-based scoring
✅ Improved alert visibility with enriched threat context
✅ Automated Slack notifications for malicious indicators
✅ Reduced alert fatigue by automatically bookmarking low-risk alerts
✅ Simulated a real-world SOC detection and response workflow

---

# Workflow Architecture




---
```mermaid
flowchart TD
    A["🔍 SigninLogs KQL Rule\nFailedAttempts >= 5"] --> B["⚠️ Sentinel Alert Created\nSeverity: HIGH"]
    B --> C["⚙️ Automation Rule\nIF alert → RUN playbook"]
    C --> D["🔄 Logic App Playbook\nAzure Workflow"]
    D --> E["📄 Parse JSON\nDecode alert entity payload"]
    E --> F["🌐 Extract IP Address\n185.220.101.1\n45.33.32.156"]
    F --> G["🛡️ VirusTotal Enrichment\n185.220.101.1 → 14/92 · malicious\n45.33.32.156 → 4/92 · low risk"]
    G --> H{Reputation\nScore Analysis}
    H -->|Malicious| I["🚨 Slack Alert\n185.220.101.1\n#soc-alerts · Block NOW"]
    H -->|Low Risk| J["🔖 Bookmark Alert\nAdded to watchlist"]

    style A fill:#0d1f35,color:#5DCAA5,stroke:#1D9E75
    style B fill:#102030,color:#B5D4F4,stroke:#185FA5
    style C fill:#0d1a2a,color:#AFA9EC,stroke:#534AB7
    style D fill:#0d1a2a,color:#FAC775,stroke:#854F0B
    style E fill:#0d1a2a,color:#D3D1C7,stroke:#5F5E5A
    style F fill:#0d1a2a,color:#EF9F27,stroke:#5F5E5A
    style G fill:#085041,color:#9FE1CB,stroke:#1D9E75
    style H fill:#0d1a2a,color:#D3D1C7,stroke:#D3D1C7
    style I fill:#7b1f12,color:#F7C1C1,stroke:#E24B4A
    style J fill:#042C53,color:#B5D4F4,stroke:#378ADD
```
# Automated Threat Intelligence Enrichment & Response Playbook
<img width="982" height="740" alt="1-playbook" src="https://github.com/user-attachments/assets/a50cb928-cfef-4279-91b0-66d643acdd5c" />



# How the Automation Works

### 1. Alert Trigger

A custom automation rule inside Microsoft Sentinel automatically triggers the Microsoft Logic App playbook whenever a security alert is generated.
<img width="1290" height="726" alt="2-Automation_rule" src="https://github.com/user-attachments/assets/8348a90d-2d1c-4954-a853-042c8ed295c5" />
Inside the Logic App playbook, an alert trigger is configured to receive alert data and initiate the automated threat enrichment and response workflow.
<img width="982" height="740" alt="1-playbook" src="https://github.com/user-attachments/assets/4b9b0f2d-4c3b-4eb7-8a48-24273494521f" />

This enables automated enrichment and response actions without requiring manual analyst intervention.

---

### 2. Data Processing

Using `Parse JSON` inside the Logic App playbook, the workflow extracts suspicious IP addresses and prepares them for threat enrichment.

This allows the automation to:

* Process alert data dynamically
* Handle multiple IP addresses
* Structure data for API communication

<img width="1186" height="692" alt="3-Parse_json" src="https://github.com/user-attachments/assets/a8d4c700-0922-4802-9ebf-bc85df695e6e" />

During testing, the playbook extracted and analyzed multiple IP addresses from the security alert:

* `185.220.101.1`
* `45.33.32.156`

The extracted IP addresses were then passed into the threat intelligence enrichment workflow for reputation analysis using VirusTotal.


---

### 3. Threat Intelligence Enrichment

The playbook sends suspicious IP addresses to the VirusTotal API using HTTP connectors.

The enrichment response includes:

* Malicious score
* Detection statistics
* Country information
* Reputation details
<img width="1225" height="687" alt="5-VT_Enrichment" src="https://github.com/user-attachments/assets/3223e2b4-9d31-440a-90b3-5f23bfef0b25" />

---

### 4. Automated Response Logic

## High-Risk Indicators

If the malicious score exceeds the defined threshold, the Logic App playbook identifies the IP address as potentially malicious and initiates an automated response workflow.

In this test scenario, the IP address `185.220.101.1` returned a malicious reputation score greater than `5`, triggering the high-risk response action.

<img width="1208" height="685" alt="6-Condition" src="https://github.com/user-attachments/assets/508d8dd4-7863-4c70-97f8-a81da400a012" />

* A detailed alert is automatically sent to Slack
* Security analysts receive enriched threat context instantly
* Helps prioritize malicious alerts faster

<img width="1197" height="646" alt="7-Send Slack Message" src="https://github.com/user-attachments/assets/23d4b3d2-b214-4e1b-b114-0c26ee1b28ff" />

### Slack Alert Message

The screenshot below shows the automated threat notification delivered to Slack after detecting the high-risk IP address `185.220.101.1`.

<img width="443" height="130" alt="7-Send Slack Message1" src="https://github.com/user-attachments/assets/12dbaa3e-7d19-4175-9f4e-db055d959c99" />

---

## Low-Risk Indicators

If the reputation score remains below the defined threshold, the Logic App playbook automatically classifies the alert as low risk and performs a bookmarking action inside Microsoft Sentinel.

In this test scenario, the IP address `45.33.32.156` returned a malicious reputation score lower than `5`, triggering the low-risk bookmarking workflow.

<img width="1161" height="697" alt="8-Bookmark" src="https://github.com/user-attachments/assets/4389911a-aa86-495b-9703-d5c6ee4e3b39" />

### Automated Alert Bookmarking

The screenshot below shows the bookmark created automatically inside Microsoft Sentinel for the low-risk IP address `45.33.32.156` to support future investigation and tracking.

<img width="1292" height="477" alt="8-Bookmark1" src="https://github.com/user-attachments/assets/03f74789-c292-4450-8301-d569f06b18ac" />

This helps:

* Reduce unnecessary escalations
* Minimize alert fatigue
* Organize low-risk findings efficiently
* Preserve investigation context for analysts


---

# Technologies Used

| Technology           | Purpose                        |
| -------------------- | ------------------------------ |
| Microsoft Sentinel   | SIEM & Alert Detection         |
| Microsoft Logic Apps | SOAR Playbook Automation       |
| VirusTotal API       | Threat Intelligence Enrichment |
| Slack                | Security Notifications         |
| HTTP Connector       | API Communication              |
| JSON Parsing         | Alert Data Processing          |

---

# Skills Demonstrated

* SOAR Automation
* SIEM Operations
* Threat Intelligence Integration
* Alert Enrichment
* Alert Triage
* Logic App Playbook Development
* API Integration
* Security Workflow Automation
* Detection & Response Engineering

---

# Future Enhancements

* MITRE ATT&CK Mapping
* Automated Severity Escalation
* Endpoint Isolation
* Email Notification Integration
* Threat Intelligence Dashboard
* Multi-source IOC Enrichment

---

# Conclusion

This project demonstrates how modern SOC teams use automation to enrich alerts, reduce repetitive investigation work, and improve response efficiency using real-world SIEM and SOAR technologies.

