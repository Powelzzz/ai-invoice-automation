# Technical Users Guide

## Prerequisites

Before building or reviewing the workflow, ensure the following are available:

- An operational `n8n` instance
- Access to Google Drive, Google Sheets, and Gmail under the same Google Workspace or Google account
- An `OCR.Space` account with a valid API key
- An OpenRouter account with API access enabled
- A sample invoice file in PDF, PNG, or JPEG format
- Permission to create and manage `n8n` credentials

## Required API Keys

The workflow depends on these secrets:

- `OCR_SPACE_API_KEY`
- `OPENROUTER_API_KEY`

Recommended environment variables are documented in [.env.example](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/.env.example).

Do not store keys directly inside workflow notes, source files, or exported JSON committed to version control.

## n8n Setup Assumptions

This guide assumes:

- `n8n` is reachable through a stable local or hosted URL
- Credential storage is enabled in `n8n`
- HTTP Request nodes can call external APIs
- The workflow runs with network access to Google APIs, `OCR.Space`, and OpenRouter
- The invoice trigger uses a dedicated Google Drive folder rather than a shared uncontrolled directory
- The workflow is designed for a demo or technical assessment, not a hardened production deployment

Operational assumptions:

- One invoice file is processed per workflow execution
- The workflow writes one row per invoice to Google Sheets
- Gmail is used for basic notifications rather than enterprise alerting

## Google Drive, Google Sheets, and Gmail Credential Requirements

### Google Drive

Use Google credentials in `n8n` with access to the monitored invoice folder.

Minimum requirements:

- Read access to the Drive folder used for invoice intake
- Permission to download file contents
- Visibility into file metadata such as file name, file ID, and created time

### Google Sheets

Use Google credentials in `n8n` with access to the target spreadsheet.

Minimum requirements:

- Access to the spreadsheet ID
- Permission to append rows
- A defined worksheet tab for invoice records

Suggested columns:

- `processed_at`
- `source_file_name`
- `vendor_name`
- `invoice_number`
- `invoice_date`
- `due_date`
- `currency`
- `subtotal`
- `tax`
- `total`
- `payment_terms`
- `status`
- `error_reason`

### Gmail

Use Gmail credentials in `n8n` with permission to send outbound emails.

Minimum requirements:

- Send access for the configured mailbox
- A valid target notification address
- Subject and body templates for success and failure scenarios

## OCR.Space Setup

Configure `OCR.Space` as an external OCR service called from an `n8n` HTTP Request node.

Required setup:

- Active `OCR.Space` API key
- Endpoint configuration using `https://api.ocr.space/parse/image`
- Input passed as either file binary or supported file URL

Recommended request considerations:

- Use the correct file source from the Google Drive download node
- Capture the raw OCR response body for debugging
- Extract the primary parsed text before passing data to OpenRouter

Failure patterns to expect:

- Unsupported file format
- Large file size
- Weak scan quality
- Empty or fragmented OCR text

## OpenRouter Setup

Configure OpenRouter as the structured extraction layer after OCR.

Required setup:

- Valid `OPENROUTER_API_KEY`
- A chosen model, for example `openai/gpt-4.1-mini`
- A constrained extraction prompt stored in [prompts/invoice_extraction_prompt.txt](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/prompts/invoice_extraction_prompt.txt)

Recommended configuration:

- Keep temperature low for stable extraction behavior
- Require JSON-only output
- Reject responses that do not match the expected schema

Expected output fields:

- `vendor_name`
- `invoice_number`
- `invoice_date`
- `due_date`
- `purchase_order_number`
- `currency`
- `subtotal`
- `tax`
- `total`
- `payment_terms`
- `line_items_summary`
- `billing_address`
- `shipping_address`
- `notes`

## Node-by-Node Data Flow

### 1. Google Drive Trigger

Purpose:
Detect a newly uploaded invoice file.

Input:
Google Drive folder event.

Output:
File metadata such as `id`, `name`, `mimeType`, and timestamps.

### 2. Google Drive Download File

Purpose:
Retrieve the file content for OCR processing.

Input:
File ID from the trigger.

Output:
Binary file content plus source metadata.

### 3. OCR.Space HTTP Request

Purpose:
Convert the invoice file into machine-readable text.

Input:
Binary file or file URL from Google Drive.

Output:
OCR API response containing parsed text.

### 4. Normalize OCR Text

Purpose:
Extract the usable text block and remove obvious OCR noise where possible.

Input:
Raw OCR response.

Output:
Normalized `ocr_text` string for the AI extraction step.

### 5. OpenRouter Extraction

Purpose:
Transform OCR text into structured invoice JSON.

Input:
Normalized OCR text and prompt instructions.

Output:
Structured invoice object.

### 6. Validate JSON

Purpose:
Confirm that required fields are present and formatted well enough for logging.

Input:
Structured invoice object.

Output:
Either a valid invoice record or an exception path payload.

### 7. Google Sheets Append Row

Purpose:
Write the processed invoice into the tracking sheet.

Input:
Validated invoice fields and metadata.

Output:
A new row in the target worksheet.

### 8. Gmail Send Notification

Purpose:
Notify stakeholders of success or failure.

Input:
Processing outcome, invoice identifier, and optional error reason.

Output:
Delivered email notification.

## Validation Logic

Minimum validation should happen before writing to Google Sheets.

Recommended checks:

- `vendor_name` is present or explicitly marked `null`
- `invoice_number` is present for traceability
- `total` is numeric
- `currency` is normalized when available
- `invoice_date` and `due_date` are in `YYYY-MM-DD` format when present
- JSON parsing succeeds without fallback guessing

Suggested decision rules:

- If `invoice_number` is missing, route to exception handling
- If `total` is missing or non-numeric, route to exception handling
- If OCR text is empty, stop early and notify failure
- If OpenRouter returns malformed JSON, retry once or route to manual review

Recommended exception payload fields:

- `source_file_name`
- `processing_stage`
- `error_reason`
- `processed_at`

## Troubleshooting

### Trigger Does Not Fire

Check that:

- The Google Drive folder ID is correct
- The connected Google account can access the folder
- The `n8n` workflow is active

### File Downloads but OCR Fails

Check that:

- The file type is supported by `OCR.Space`
- The HTTP Request node is sending the correct binary property
- The API key is valid
- The file is not image-corrupted or unreadable

### OCR Returns Text but AI Output Is Poor

Check that:

- The OCR text is not heavily fragmented
- The prompt is constrained and JSON-only
- The selected model is appropriate for structured extraction
- Temperature is set low

### OpenRouter Returns Invalid JSON

Check that:

- The prompt explicitly forbids markdown and explanations
- The workflow parses only the assistant content, not wrapper metadata
- The model response is validated before downstream use

### Google Sheets Write Fails

Check that:

- The spreadsheet ID is correct
- The worksheet name exists
- The credential has append permission
- The field mapping order matches the sheet columns

### Gmail Notification Fails

Check that:

- Gmail credentials are valid
- The sender account is authorized to send mail
- The target recipient address is valid

### Duplicate Invoices Appear

Likely causes:

- The same file was uploaded twice
- The trigger replayed an event
- There is no deduplication check on `invoice_number` plus source file name

Recommended mitigation:

- Add an idempotency check before appending to Google Sheets

## Reference Files

- [README.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/README.md)
- [docs/architecture.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/docs/architecture.md)
- [workflow/README.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/README.md)
- [workflow/n8n-workflow-notes.md](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/workflow/n8n-workflow-notes.md)
- [prompts/invoice_extraction_prompt.txt](/Users/lawrencepaulabayata/Projects/ai-invoice-automation/prompts/invoice_extraction_prompt.txt)
