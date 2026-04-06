# n8n Workflow Notes

## Overview

This workflow is named **AI Invoice Processing and Accounting Entry Automation**. It has nine active nodes plus a sticky note import checklist. The success and manual review branches are visually separated.

---

## Node-by-Node Notes

### 1. Google Drive Trigger

- Type: Google Drive Trigger
- Event: `fileCreated` on a specific folder
- Poll interval: every minute (suitable for demo; increase for production)
- Watch folder: set via `GOOGLE_DRIVE_FOLDER_ID`
- Outputs: `id`, `name`, `mimeType`, `createdTime`, `webViewLink`
- **Assumption**: Only the designated `/Incoming Invoices` folder is watched. Files placed in subfolders are not captured.

### 2. Download Invoice

- Type: Google Drive — Download File
- Input: `{{ $json.id }}` from the trigger
- Output: binary file content in the `data` property
- **Assumption**: The file is a readable PDF or image. Password-protected files will fail OCR silently.

### 3. OCR.Space Extract

- Type: HTTP Request (POST)
- Endpoint: `https://api.ocr.space/parse/image`
- Auth: `apikey` header set to `OCR_SPACE_API_KEY`
- Body: multipart form-data with `file` (binary), `language=eng`, `OCREngine=2`, `isOverlayRequired=false`
- OCR Engine 2 is used because it performs better on printed documents and invoices.
- **Assumption**: File is sent as binary (`data` property). If the MIME type is unsupported, the OCR response will be empty.

### 4. Normalize OCR Text

- Type: Code (JavaScript, runs once per item)
- Reads `ParsedResults[0].ParsedText` from the OCR response
- Re-attaches trigger metadata: `source_file_id`, `source_file_name`, `source_file_mime_type`, `source_file_created_time`, `source_file_link`
- Output field: `ocr_text` — used by the OpenRouter prompt
- **Assumption**: OCR always returns at least one `ParsedResults` entry. Empty text is passed downstream and caught by validation.

### 5. OpenRouter Extract Fields

- Type: HTTP Request (POST)
- Endpoint: `https://api.openrouter.com/v1/chat/completions`
- Model: `openai/gpt-4.1-mini` (balance of cost and quality; swap to `openai/gpt-4o` for higher accuracy)
- `response_format`: `{ "type": "json_object" }` forces JSON-only output
- Temperature: default (0.0 is ideal for structured extraction; set via model config if needed)
- Prompt: inlined from `prompts/invoice_extraction_prompt.txt`
- **Assumption**: OpenRouter returns a single valid JSON object. The next node handles parse failures.

### 6. Parse Structured Invoice

- Type: Code (JavaScript, runs once per item)
- Parses the assistant `choices[0].message.content` as JSON
- Normalizes numeric fields (`subtotal`, `tax`, `total_amount`) — strips symbols, coerces to Number
- Checks for missing required fields: `vendor_name`, `invoice_number`, `total_amount`
- Checks `confidence_notes` for uncertainty keywords: `uncertain`, `low confidence`, `missing`, `unclear`, `not found`
- Sets `is_valid: true/false`, `status`, `review_needed`, and `validation_errors`
- **Assumption**: All downstream nodes read from this node's output. It is the single source of truth for field values.

### 7. Validation

- Type: IF node
- Condition: `{{ $json.is_valid }} === true`
- `true` branch (output 0) → success path
- `false` branch (output 1) → manual review path
- **Assumption**: Any error in JSON parsing, missing fields, or flagged confidence notes sends the item to manual review.

### 8a. Append Success Row *(success path)*

- Type: Google Sheets — Append Row
- Sheet: `GOOGLE_SHEET_ID`, tab `Sheet1`
- Maps all 15 columns: Upload Timestamp, File Name, Vendor, Invoice Number, Invoice Date, Due Date, Currency, Subtotal, Tax, Total Amount, Suggested Category, Status, Review Needed, Source File Link, Confidence Notes
- Status is written as `Ready for review`
- **Assumption**: Sheet columns exist in the exact order listed. Mismatched column names cause silent blank writes.

### 8b. Append Manual Review Row *(manual review path)*

- Same sheet and column structure as the success path
- Status hardcoded as `Manual Review`, Review Needed as `Yes`
- Confidence Notes field writes `validation_errors` (or falls back to `confidence_notes`)
- **Assumption**: Finance team monitors the sheet and filters by `Review Needed = Yes`.

### 9a. Send Success Email *(success path)*

- Type: Gmail — Send
- Subject: `New invoice processed: {{ invoice_number }}`
- Body includes: vendor, invoice number, total + currency, status, file name
- `sendTo`: set to `GMAIL_NOTIFICATION_TO` value

### 9b. Send Manual Review Email *(manual review path)*

- Subject: `Manual review required: {{ source_file_name }}`
- Body includes: file name, vendor (if known), invoice number (if known), failure reason, source file link
- **Assumption**: Email is the only alerting channel. For production, a Slack or ticketing integration would replace or supplement this.

---

## Credential Setup

| Service | n8n Credential Type | Notes |
|---|---|---|
| Google Drive | Google OAuth2 | Used by trigger and download nodes |
| Google Sheets | Google OAuth2 | Same credential is fine |
| Gmail | Google OAuth2 | Requires Gmail send scope |
| OCR.Space | None (header key) | Set `apikey` directly in the HTTP Request node |
| OpenRouter | None (header key) | Set `Authorization: Bearer ...` in the HTTP Request node |

---

## Good Assessment Practices

- Each node is named clearly: no default `HTTP Request 1` names.
- The success path and exception path are visually separated vertically.
- The sticky note at the top left lists the import checklist for quick setup.
- Add brief inline notes in n8n (`Node Notes` field) where assumptions matter most: OCR Engine choice, OpenRouter model, and sheet column order.
