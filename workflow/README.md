# Workflow Guide

This folder documents the intended `n8n` workflow design for the invoice processing assessment.

## Importable Starter

Use [invoice_automation_n8n.json](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/invoice_automation_n8n.json) as the starter workflow import.

After import, replace the placeholder values for:

- Google Drive folder ID
- Google Sheet ID and sheet tab name
- `OCR.Space` API key
- OpenAI API key
- Gmail recipient email
- Google node credentials

## Workflow Objective

Convert incoming invoice files into structured records and notify stakeholders with minimal manual intervention.

## Suggested High-Level Node Layout

1. Google Drive Trigger
2. Download File
3. OCR.Space Request
4. Normalize OCR Text
5. OpenAI Extraction
6. Validate JSON
7. Append to Google Sheets
8. Send Gmail Notification

## Outputs

- Processed invoice row in Google Sheets
- Notification email
- Exception visibility for failed cases

Use [workflow/n8n-workflow-notes.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/n8n-workflow-notes.md) for implementation details and assumptions.
