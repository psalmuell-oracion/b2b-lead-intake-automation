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

Data Pre-Processing & Reference Logic Proof
Figure 1: Google Sheets Dynamic Pre-Processing (Leads Tab)
[01_GoogleSheets_Leads_Preprocessed.png]

Figure 2: Relational Reference Data Schema (Reference_Data Tab)
[02_GoogleSheets_ReferenceData_LookupTable.png]

Figure 3: SQL-Style Real-Time Executive View (Hot_Leads_Dashboard Tab)
[03_GoogleSheets_HotLeads_Dashboard_Query.png]

2. Automation Architecture & Integration (Zapier)
The pipeline utilizes a multi-step Zapier workflow configured with strict conditional filters to protect CRM hygiene and focus sales execution on high-converting leads.

Workflow Configurations
Trigger — Google Sheets: Monitors New or Updated Spreadsheet Row in the Leads sheet.

Control Logic — Filter by Zapier: Evaluates Lead Rating. Only proceeds if Lead Rating (Text) Exactly matches Hot. Stops execution for Warm or Cold leads.

Action — Salesforce (Create Lead): Connects via OAuth (Production Endpoint). Maps pre-processed Google Sheets attributes into Salesforce Lead object fields.

Action — Gmail (Send Email): Dispatches an instant plain-text SLA alert email to account managers containing lead parameters (Name, Company, Email, SLA Target, Action Priority).

Integration & Execution Layer Proof
Figure 4: Multi-Step Zapier Automation Canvas
[04_Zapier_MultiStep_Automation_Canvas.png]

Figure 5: Salesforce CRM Ingested Record View
[05_Salesforce_CRM_Ingested_Leads_View.png]

Figure 6: Executive SLA Gmail Alert Dispatch
[06_Gmail_Executive_SLA_Notification.png]

3. Verification & Scenario Test Results
Scenario Test Record: Rachel Vance | Vanguard Analytics | r.vance@vanguardanalytics.io | Hot

Pre-Processing Output: Successfully calculated SLA Target (15-Min Response), Domain (vanguardanalytics.io), and Action Priority (Priority 1 - Direct Call).

Dashboard Output: Dynamically pulled into Hot_Leads_Dashboard via QUERY.

CRM Output: Created clean Lead Record in Salesforce CRM under My Leads.

Alert Output: Fired plain-text email alert with non-null field attributes.
