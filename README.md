# 🚀 Production-Grade AI Newsletter System (Moringa News)

## 📌 Project Overview
This repository contains the n8n deployment files for the Moringa News Automated Newsletter System. Upgraded from a functional prototype to a hardened, production-grade architecture, this system features parallel data ingestion, AI-driven content curation, advanced human-in-the-loop feedback cycles, and a dedicated error-handling pipeline. 

The architecture strictly adheres to enterprise standards: Structured AI Outputs, Fault Tolerance, Intentional Model Selection, and Operational Governance.

## 🏗️ System Architecture & Node Breakdown
The system is decoupled into three distinct workflows to ensure modularity, scalability, and fault tolerance:

### 1. Data Ingestion Pipeline (`Data Ingestion Pipeline.json`)
**Purpose:** Safely extract, standardize, and clean raw data from multiple sources.
* **Trigger:** Scheduled to run automatically at 2:00 PM every Friday.
* **Parallel Extraction:** Scrapes 5 distinct RSS feeds (The Standard, BBC Business, BBC Technology, Nairobi Events, ESPN).
* **Data Standardization:** Utilizes `Limit` nodes to control token usage and `Set` nodes to standardize the payload structure across all 5 disparate sources.
* **Pre-Processing Logging:** Logs raw fetched data to Google Sheets immediately to ensure data provenance.
* **AI Data Cleaner:** An AI Agent (Mistral Primary + OpenRouter Fallback) cleans the aggregated data. It is constrained by a `Structured Output Parser` to guarantee downstream JSON reliability.
* **Handoff:** Uses the `Execute Workflow` node to securely pass the cleaned payload to the Newsletter Generator.

### 2. Newsletter Generator (`Newsletter Generator.json`)
**Purpose:** Synthesize the cleaned data, fetch audience metrics, and manage human approval and revisions.
* **Trigger:** `When Executed by Another Workflow` (receives data from the Ingestion Pipeline).
* **Audience Integration:** Reads dynamic subscriber data from Google Sheets and extracts emails for targeted delivery.
* **Chief Editor AI:** Synthesizes the initial newsletter draft. Configured with strict prompt constraints and a Fallback Model to prevent timeouts.
* **Human-in-the-Loop (HITL) & Feedback Loop:** Sends a draft via Gmail (`sendAndWait`) and pauses execution using a `Wait` node. A `Decision` node then evaluates the human response: if **approved**, the draft bypasses further editing and proceeds directly to delivery. If **disapproved** with editorial feedback, a **Backup Editor AI** dynamically reaches back across the workflow to grab the original raw stories and completely rewrites the HTML draft based on the strict human constraints.
* **Delivery & Logging:** Dispatches the final HTML newsletter, saves the final log to the database, and notifies the team.

### 3. Error Workflow (`Error Workflow.json`)
**Purpose:** A dedicated, system-wide listener that catches catastrophic failures and prevents silent crashes.
* **Trigger:** `Error Trigger` (Listens to all published workflows in the n8n instance).
* **AI Error Triage:** Passes raw error data to an Error Analysis Agent.
* **Structured Enforcement:** A `Structured Output Parser` forces the AI to output a strict JSON schema containing a `Solution` and a `Priority_Level` (High/Medium/Low).
* **Governance Logging:** Logs highly actionable data to Google Sheets, including the exact `RunID`, Node Name, Timestamp, and AI Solution for complete auditability.
* **Priority Routing:** A `Switch` node routes the alert to the Engineering Team via Gmail based on the AI-determined priority.

## ⚙️ Deployment Instructions
1. Import all three `.json` files into your n8n instance.
2. Update Credentials for Google Sheets, Gmail, and OpenRouter/Mistral across all workflows.
3. **Crucial:** Ensure ALL THREE workflows are set to `Published` (Active). The system relies on active triggers (Schedule, Execute by Another, and Error Trigger) to function autonomously.
4. Execute the Data Ingestion Pipeline to initiate the full system run.
