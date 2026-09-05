End-to-End Lead Capture and Qualification Workflow

## Project Overview

This project implements an automated lead management system using **n8n**. It captures incoming leads, validates and cleans the data, prevents duplicate entries, analyzes lead intent using AI, classifies leads into Hot, Warm, or Cold categories, stores the results in Google Sheets, and performs category-specific actions.

The project also includes a separate **Daily Lead Summary Report** workflow that runs every day at 9:00 AM and emails a summary of the leads received during the day.

---

## Project Objectives

The workflows are designed to meet the following objectives:

- Capture lead submissions through a webhook.
- Validate and clean incoming lead data.
- Handle missing or invalid data.
- Detect duplicate leads.
- Perform AI-based lead intent analysis.
- Classify leads into Hot, Warm, and Cold tiers.
- Store all qualified leads in Google Sheets.
- Send immediate notifications for Hot leads.
- Add Warm leads to a follow-up queue.
- Archive Cold leads.
- Generate a daily summary report.

---

# Workflow 1: End-to-End Lead Capture and Qualification

## Workflow Overview

The main workflow processes leads from initial submission to final qualification and notification.

### Workflow Flow

```text
Lead Capture Webhook
        ↓
Validate & Clean Data
        ↓
      Is Valid?
     ↙        ↘
Invalid      Lookup Existing Lead
   ↓               ↓
Respond          Check Duplicate
Invalid              ↓
                 Is Duplicate?
                ↙          ↘
          Duplicate      AI Lead Intent Analysis
              ↓                 ↓
       Respond Duplicate    Classify Lead Tier
                                    ↓
                            Append to Master Log
                                    ↓
                                Is Hot Lead?
                               ↙          ↘
                            Yes            No
                             ↓              ↓
                     Send Email +      Is Warm Lead?
                     Slack Alert       ↙         ↘
                                  Warm          Cold
                                    ↓             ↓
                           Follow-up Queue     Archive
```

---

## Nodes and Logic

### 1. Lead Capture Webhook

Receives lead data submitted to the workflow.

Expected lead information includes fields such as:

- Name
- Email
- Phone
- Company
- Budget
- Source

This node acts as the entry point for the automation.

---

### 2. Validate & Clean Data

This node cleans and validates the incoming lead information.

The validation process checks for:

- Required fields
- Valid email format
- Empty or missing values
- Data consistency

Invalid records are sent to the **Respond Invalid** branch.

---

### 3. Is Valid?

An IF node determines whether the incoming lead data passed validation.

- **True:** Continue processing the lead.
- **False:** Return an invalid response.

---

### 4. Lookup Existing Lead

The workflow checks the Google Sheets master log to determine whether the lead already exists.

This helps prevent duplicate records.

---

### 5. Check Duplicate

The workflow evaluates the lookup result and determines whether the current lead is already present in the system.

---

### 6. Is Duplicate?

The IF node routes the lead based on the duplicate check.

- **True:** The lead is handled by the **Respond Duplicate** node.
- **False:** The lead continues to AI analysis.

---

### 7. AI Lead Intent Analysis

An **AI Agent** with a **Google Gemini Chat Model** analyzes the lead.

The AI analysis evaluates the lead's likely intent and quality based on the available information.

A **Structured Output Parser** is connected to ensure that the AI returns data in a consistent format.

The structured output includes information such as:

- Intent score
- Intent level
- Analysis or reasoning

---

### 8. Classify Lead Tier

Based on the available lead data and AI analysis, the workflow assigns the lead to an appropriate category:

#### Hot Lead
High-value or high-intent leads.

Typical characteristics include:

- High budget
- Strong purchase intent
- Corporate email domain
- Good-quality lead information

#### Warm Lead
Potential leads that require follow-up.

Typical characteristics include:

- Medium budget
- Good data completeness
- Moderate intent

#### Cold Lead
Low-priority leads.

Typical characteristics include:

- Low budget
- Missing information
- Free or personal email domains
- Low intent

---

### 9. Append to Master Log

All processed leads are stored in the Google Sheets master log.

The stored information includes the lead details and qualification status.

---

### 10. Is Hot Lead?

The workflow checks whether the classified lead is a Hot lead.

If the result is true, the lead receives immediate notifications.

---

### 11. Send Email to Sales

An email notification is sent to the sales team containing the important lead information.

This ensures that high-priority leads receive immediate attention.

---

### 12. Slack Notify Sales

A Slack notification is sent to the configured sales channel for Hot leads.

The notification includes lead details such as:

- Name
- Company
- Budget
- Email
- AI intent information

---

### 13. Is Warm Lead?

Leads that are not Hot are checked to determine whether they are Warm.

- **Warm:** Added to the follow-up queue.
- **Cold:** Sent to the archive.

---

### 14. Append to Follow-up Queue

Warm leads are stored in a dedicated follow-up queue.

This allows the sales team to contact and nurture potential customers later.

---

### 15. Append to Archive

Cold leads are stored in an archive for future reference.

---

### 16. Response Nodes

The workflow includes response handling for:

- Invalid leads
- Duplicate leads
- Successfully processed leads

This provides appropriate feedback for each workflow outcome.

---

# Workflow 2: Daily Lead Summary Report

## Workflow Overview

The second workflow automatically generates a daily summary of the leads stored in the master log.

### Workflow Flow

```text
Every Day at 9:00 AM
        ↓
Read Master Log
        ↓
Aggregate Counts
        ↓
Email Daily Summary
```

---

## Nodes and Logic

### 1. Every Day 9AM

A Schedule Trigger starts the workflow automatically every day at **9:00 AM**.

---

### 2. Read Master Log

The workflow reads lead records from the Google Sheets master log.

---

### 3. Aggregate Counts

A Code node processes the lead records and calculates:

- Total number of new leads
- Number of Hot leads
- Number of Warm leads
- Number of Cold leads

The workflow generates a summary message containing these counts.

---

### 4. Email Daily Summary

The generated summary is sent through email.

The email provides a quick overview of the day's lead activity.

Example summary:

```text
Daily Lead Summary

Total new leads: X
Hot: X
Warm: X
Cold: X
```

---

# Technologies Used

- **n8n** – Workflow automation
- **Webhook** – Lead data capture
- **Google Sheets** – Lead storage and management
- **Google Gemini Chat Model** – AI-based lead intent analysis
- **AI Agent** – Lead analysis
- **Structured Output Parser** – Consistent AI output
- **IF Nodes** – Conditional routing
- **Code Nodes** – Data validation, processing, and aggregation
- **Email/SMTP** – Email notifications and reports
- **Slack** – Instant notifications for Hot leads

---

# Key Features

## Data Validation

Incoming lead information is checked before further processing to handle:

- Missing required fields
- Invalid data
- Invalid email formats

## Duplicate Prevention

Existing leads are checked before being stored to avoid duplicate records.

## AI-Based Lead Scoring

The AI Agent analyzes lead information and provides structured lead intent data.

## Multi-Tier Classification

Leads are classified into:

- Hot
- Warm
- Cold

## Targeted Notifications

Different actions are performed based on lead quality:

| Lead Tier | Action |
|---|---|
| Hot | Store in Master Log + Email + Slack notification |
| Warm | Store in Master Log + Add to Follow-up Queue |
| Cold | Store in Master Log + Archive |

## Daily Reporting

A separate scheduled workflow generates daily counts for each lead category.

---

# Project Structure

```text
Assignment 10/
│
├── End-to-End Lead Capture and Qualification Workflow.json
│
├── Daily Lead Summary Report.json
│
├── README.md
│
└── Screenshots/
    ├── Main Workflow Canvas
    ├── AI Agent Configuration
    ├── Structured Output Configuration
    └── Execution Results
```

---

# How to Import the Workflows

1. Open your n8n instance.
2. Create or open a workflow.
3. Use the **Import from File** option.
4. Select the required JSON workflow file.
5. Configure your own credentials for:
   - Google Sheets
   - Google Gemini
   - Slack
   - Email/SMTP
6. Update sheet names, notification channels, and email addresses if required.
7. Test the workflow before activating it.

---

# Testing

The main workflow should be tested with multiple scenarios:

### Valid Hot Lead
Verify that:

- The lead passes validation.
- The lead is not a duplicate.
- The lead is classified correctly.
- The lead is stored in the master log.
- Email and Slack notifications are sent.

### Warm Lead
Verify that the lead is added to the follow-up queue.

### Cold Lead
Verify that the lead is stored in the archive.

### Invalid Lead
Verify that invalid or incomplete data is handled correctly.

### Duplicate Lead
Verify that an existing lead is not stored again.

### Daily Summary
Verify that the daily workflow reads the master log, calculates category counts, and sends the summary email.

---


# Notes

Credentials are not included as usable secrets in exported workflow JSON files. After importing the workflows into another n8n instance, credentials may need to be configured again.

Sensitive information such as personal email addresses, spreadsheet identifiers, and API credentials should be reviewed before sharing the workflow publicly.
