# 🚀 Automated Identity Threat Response (SOAR) Pipeline

## 📌 Executive Summary
In modern cloud environments, speed is the ultimate deciding factor during an identity compromise. Relying on a human SOC analyst to manually review an alert and revoke access gives an attacker too much time to pivot.

To reduce the Mean Time to Respond (MTTR) from minutes to milliseconds, I engineered a fully automated Security Orchestration, Automation, and Response (SOAR) pipeline using **Microsoft Sentinel** and **Azure Logic Apps**. This project demonstrates a zero-touch remediation workflow that instantly isolates compromised Microsoft Entra ID accounts.

---

## 🏗️ Architecture Flow

1. **Detection:** Microsoft Entra ID flags an anomalous "Impossible Travel" login.
2. **Alerting:** Microsoft Sentinel ingests the logs and generates a High-Risk Incident.
3. **Trigger:** A Sentinel Automation Rule instantly fires an Azure Logic App playbook.
4. **Execution:** The Logic App parses the JSON payload, extracts the compromised Entra ID user, and sends a `POST` request via the Microsoft Graph API.
5. **Containment:** All active session and refresh tokens for the compromised user are instantly revoked.
6. **Notification:** An automated HTML email is sent via SMTP to alert the SOC team.

---

## 🛠️ Tools & Technologies
* **SIEM:** Microsoft Sentinel (Unified Defender Portal)
* **SOAR:** Azure Logic Apps
* **Identity Provider:** Microsoft Entra ID
* **Automation Interface:** Microsoft Graph API (RESTful POST requests)
* **Core Concepts:** Role-Based Access Control (RBAC), OAuth 2.0 App Registrations, Zero Trust Architecture, JSON Payload Parsing.

---

## ⚙️ Step-by-Step Build Guide

### Step 1: Configuring the SIEM Trigger
The foundation of the automation is the **Microsoft Sentinel Incident** trigger. When an incident is created, Sentinel passes a massive JSON array containing all the metadata about the attack. 

### Step 2: The Logic App & Data Parsing
Because a single incident can involve multiple compromised accounts, I implemented an **Entities - Get Accounts** helper action followed by a **For each** loop. This ensures the SOAR platform dynamically scales to isolate every single compromised identity found in the alert payload.


### Step 3: The Graph API Remediation (The Muscle)
To forcefully kick the attacker out of the environment, the Logic App makes an authenticated HTTP call to the Microsoft Graph API using an OAuth 2.0 Service Principal.

---

## 🚧 Overcoming Real-World Engineering Hurdles
Building this pipeline revealed several strict architectural requirements standard tutorials often miss.

### 1. API Routing Strictness (404 Not Found & 400 Bad Request)
Initially, the Graph API rejected the `POST` commands. Through raw JSON log analysis, I identified two critical API rules:
* **Endpoint Targeting:** You cannot pass a display name into the URI; the API requires the exact alphanumeric Entra ID Object ID to execute the session revocation.
* **Formatting Strictness:** Even a single blank space in the dynamic URI string will break the web request. The payload requires a strict `Content-Type: application/json` header and an empty `{}` JSON body.

### 2. The "Phantom" RBAC Permission
Even as an Azure Global Administrator, Sentinel initially refused to trigger the playbook, throwing a "No Microsoft Sentinel permissions" error. This occurs because the Microsoft Sentinel background service account (*Azure Security Insights*) requires explicit authority to reach into other resource groups. I resolved this by manually assigning the **Microsoft Sentinel Automation Contributor** role to the specific Resource Group, securely bridging the SIEM and SOAR tools based on the principle of least privilege.


---

## 🎯 Final Outcome
By successfully connecting the SIEM directly to the Identity Provider via API automation, this project entirely removes the human element from the immediate containment phase. The threat actor is forcefully logged out of all active Microsoft sessions before an analyst even opens the ticket.

---
*Note: The raw JSON workflow code for this Logic App is included in the `src/` folder of this repository.*
