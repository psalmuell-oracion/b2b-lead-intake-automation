# B2B Lead Intake & SLA Routing System

**Project Type:** End-to-End Data Pipeline & Operations Automation  
**Tech Stack:** Google Sheets, Zapier, Salesforce CRM (Developer Org), Gmail API  
**Core Stack Logic:** `ARRAYFORMULA`, `XLOOKUP`, `IFS`, `QUERY`, `RIGHT`, `LEN`, `FIND`, `TRIM`, Zapier Webhooks, OAuth 2.0  
**Live Data Pipeline (Read-Only):** [View Google Sheet Tracker](https://docs.google.com/spreadsheets/d/14vfo6GTqu_nmBvITRfyz0X58tEUVdYCBRGFmFqVfuL4/edit?usp=sharing)

---

## Executive Summary & Architecture

This project automates the B2B lead intake process, transforming raw form entries into enriched CRM records and real-time executive alerts. By applying dynamic data engineering formulas inside Google Sheets before triggering Zapier automation, the system eliminates manual data entry, reduces error rates, and ensures high-priority (`Hot`) leads trigger immediate sales SLAs.

### Workflow Pipeline
`[ Raw Google Sheet Entry ]` ──► `[ Dynamic Pre-Processing & SQL Filtering ]` ──► `[ Zapier Filter Logic ]` ──► `[ Salesforce Lead Creation ]` ──► `[ Executive Gmail Alert ]`

---

## 1. Data Engineering & Pre-Processing (Google Sheets)

Before any data reaches the CRM, raw input fields (`First Name`, `Last Name`, `Email`, `Lead Rating`) undergo multi-layer calculations to standardize lead profiles.

### Technical Formula Implementations

* **Dynamic Full Name Concatenation (`Column C`):**
  ```excel
  =ARRAYFORMULA(IF(ISBLANK(A2:A), "", TRIM(A2:A & " " & B2:B)))
Automates full name merging down the sheet without requiring manual formula drag-downs.

Corporate Domain Extraction (Column J):

Excel
=IF(ISBLANK(E2), "", RIGHT(E2, LEN(E2) - FIND("@", E2)))
Isolates domain strings for account-level matching and organizational grouping.

Relational SLA Lookup (Column I):

Excel
=IFERROR(XLOOKUP(H2, Reference_Data!A:A, Reference_Data!B:B, "Unassigned"), "")
Maps Lead Rating against the target response window defined in the relational lookup sheet.

Operational Action Priority (Column L):

Excel
=IFS(H2="Hot", "Priority 1 - Direct Call", H2="Warm", "Priority 2 - Email Sequence", H2="Cold", "Priority 3 - Monthly Newsletter", TRUE, "Unassigned")
Assigns immediate action protocols based on lead urgency.

Real-Time Executive SQL Reporting (Hot_Leads_Dashboard Tab):

Excel
=QUERY(Leads!A2:L, "SELECT C, D, E, H, I, L WHERE H = 'Hot' AND K = 'Pending' ORDER BY C ASC", 0)
Filters and displays pending high-priority prospects for executive visibility.

## Data Pre-Processing & Reference Logic Proof

Figure 1: Google Sheets Dynamic Pre-Processing (Leads Tab)
<img width="1366" height="768" alt="01_GoogleSheets_Leads_Preprocessed" src="https://github.com/user-attachments/assets/8aad4e3d-038b-4a52-bce2-c4b6c2217d1b" />


Figure 2: Relational Reference Data Schema (Reference_Data Tab)
<img width="1366" height="768" alt="02_GoogleSheets_ReferenceData_LookupTable" src="https://github.com/user-attachments/assets/b000434c-69d3-4dc7-905a-bdb31e38ae12" />


Figure 3: SQL-Style Real-Time Executive View (Hot_Leads_Dashboard Tab)
<img width="1366" height="768" alt="03_GoogleSheets_HotLeads_Dashboard_Query" src="https://github.com/user-attachments/assets/3e18ccf2-45ef-40da-a377-eab864571a2b" />

---

## 2. Automation Architecture & Integration (Zapier)

The pipeline utilizes a multi-step Zapier workflow configured with strict conditional filters to protect CRM hygiene and focus sales execution on high-converting leads.

Workflow Configurations
Trigger — Google Sheets: Monitors New or Updated Spreadsheet Row in the Leads sheet.

Control Logic — Filter by Zapier: Evaluates Lead Rating. Only proceeds if Lead Rating (Text) Exactly matches Hot. Stops execution for Warm or Cold leads.

Action — Salesforce (Create Lead): Connects via OAuth (Production Endpoint). Maps pre-processed Google Sheets attributes into Salesforce Lead object fields.

Action — Gmail (Send Email): Dispatches an instant plain-text SLA alert email to account managers containing lead parameters (Name, Company, Email, SLA Target, Action Priority).

## Integration & Execution Layer Proof

Figure 4: Multi-Step Zapier Automation Canvas
<img width="1366" height="768" alt="04_Zapier_MultiStep_Automation_Canvas" src="https://github.com/user-attachments/assets/be2b86ca-8d8f-479e-a1b3-2482f230b6bc" />


Figure 5: Salesforce CRM Ingested Record View
<img width="1366" height="768" alt="05_Salesforce_CRM_Ingested_Leads_View" src="https://github.com/user-attachments/assets/91e0f1b6-3a2b-4821-a3c1-8dfc796669f7" />


Figure 6: Executive SLA Gmail Alert Dispatch
<img width="1366" height="768" alt="06_Gmail_Executive_SLA_Notification" src="https://github.com/user-attachments/assets/ce9037f8-dcad-4590-a9a6-ca7541d8f822" />

---

## 3. Verification & Scenario Test Results

Scenario Test Record: Rachel Vance | Vanguard Analytics | r.vance@vanguardanalytics.io | Hot

Pre-Processing Output: Successfully calculated SLA Target (15-Min Response), Domain (vanguardanalytics.io), and Action Priority (Priority 1 - Direct Call).

Dashboard Output: Dynamically pulled into Hot_Leads_Dashboard via QUERY.

CRM Output: Created clean Lead Record in Salesforce CRM under My Leads.

Alert Output: Fired plain-text email alert with non-null field attributes.
