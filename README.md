# AI Invoice Processing and Accounting Automation

## Overview

This project presents an interview-ready design for automating invoice intake, extraction, validation, accounting handoff, and stakeholder notification. The solution uses `n8n` as the orchestration layer and connects `Google Drive`, `OCR.Space`, `OpenAI`, `Google Sheets`, and `Gmail` into a practical finance operations workflow.

The objective is to reduce manual invoice handling, improve data consistency, and create a simple but credible automation pattern that can be extended into a production-grade accounting process.

## Business Challenge

Finance teams often receive invoices across multiple suppliers, formats, and scan qualities. Important fields such as vendor name, invoice number, dates, totals, tax, and payment terms must be captured accurately and routed into a tracking or accounting process.

In many small and mid-sized operations, this work is still handled manually. That creates delays, inconsistent records, avoidable entry errors, and poor visibility into invoice status.

## Why Manual Invoice Processing Is Inefficient

- It is repetitive and time-consuming.
- Data entry errors can lead to payment delays and reporting issues.
- Invoice formats vary widely, making standardization difficult.
- Manual reviews slow down turnaround time for finance teams.
- Exception handling is inconsistent when there is no structured workflow.
- Operational visibility is weak when invoice status is tracked informally.

## Tools Used

- `n8n`: workflow orchestration and integration logic
- `Google Drive`: invoice intake and file storage trigger point
- `OCR.Space`: OCR extraction for invoice PDFs and images
- `OpenAI`: structured invoice field extraction and normalization
- `Google Sheets`: lightweight invoice log and review surface
- `Gmail`: stakeholder notifications for success and exception cases

## Architecture Overview

The solution follows an event-driven pattern:

1. A new invoice is uploaded to a designated Google Drive folder.
2. `n8n` detects the new file and downloads it.
3. `OCR.Space` extracts raw text from the invoice.
4. `OpenAI` converts OCR text into structured invoice JSON.
5. The workflow validates required fields and handles exceptions.
6. `Google Sheets` records the processed invoice.
7. `Gmail` sends a notification confirming success or flagging an issue.

This architecture keeps the flow easy to explain during an assessment while still demonstrating practical automation design, AI-assisted extraction, and downstream business integration.

## Workflow Summary

### Intake

Invoices enter the process through a monitored Google Drive folder.

### Extraction

The file content is passed to `OCR.Space` to retrieve machine-readable text.

### Structuring

The OCR output is sent to `OpenAI` with a constrained extraction prompt that returns normalized invoice fields in JSON format.

### Validation

The workflow checks that mandatory fields such as vendor name, invoice number, and total amount are present before proceeding.

### Recording

Structured results are written into Google Sheets as a lightweight accounting operations register.

### Notification

Gmail sends a success or exception email so stakeholders know whether the invoice was processed cleanly or needs review.

## Setup Guide

### 1. Prerequisites

- Access to an `n8n` environment
- A Google account with Drive, Sheets, and Gmail access
- An `OCR.Space` API key
- An OpenAI API key

### 2. Configure Environment Variables

Use [.env.example](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/.env.example) as the baseline configuration file.

Required values include:

- `OPENAI_API_KEY`
- `OPENAI_MODEL`
- `OCR_SPACE_API_KEY`
- `GOOGLE_DRIVE_FOLDER_ID`
- `GOOGLE_SHEET_ID`
- `GMAIL_SENDER`
- `GMAIL_NOTIFICATION_TO`

### 3. Create the Supporting Assets

- Create the Google Drive folder that will receive invoices.
- Create the target Google Sheet for invoice records.
- Configure Gmail and Google credentials inside `n8n`.

### 4. Build or Import the Workflow

Use the notes in [workflow/README.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/README.md) and [workflow/n8n-workflow-notes.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/n8n-workflow-notes.md) to assemble the workflow.

Suggested node order:

1. Google Drive Trigger
2. Download File
3. OCR.Space HTTP Request
4. Normalize OCR Text
5. OpenAI Extraction
6. Validate JSON
7. Append Row to Google Sheets
8. Send Gmail Notification

### 5. Test with a Sample Payload

Use [scripts/sample_invoice_payload.json](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/scripts/sample_invoice_payload.json) and [prompts/invoice_extraction_prompt.txt](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/prompts/invoice_extraction_prompt.txt) to validate the extraction logic before running live files.

## KPIs

The following KPIs are appropriate for this automation:

- Invoice processing turnaround time
- OCR-to-structured-data accuracy
- Percentage of invoices processed without manual intervention
- Exception rate
- Average time spent on manual review
- Reduction in manual data entry effort

For an assessment setting, these KPIs demonstrate both business awareness and technical maturity.

## Live Demo Summary

The live demo should show a complete invoice journey from file upload to recorded output:

1. Upload a sample invoice into Google Drive.
2. Show `n8n` detecting and processing the file.
3. Highlight OCR extraction and AI-based field structuring.
4. Show the final record in Google Sheets.
5. Show the Gmail notification for process completion.
6. Briefly explain how exception handling would capture poor OCR or missing required fields.

This demo format is effective for a technical assessment because it clearly demonstrates orchestration, third-party integrations, AI-assisted document extraction, and business value in a short timeframe.
