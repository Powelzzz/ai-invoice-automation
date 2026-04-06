# Demo Script

## Goal

Walk the panel through a complete invoice automation journey in five to seven minutes. Be conversational — explain what you see on screen as it happens.

---

## Before You Start

- Have Google Drive open in one tab with the `/Incoming Invoices` folder visible.
- Have the n8n workflow open in a second tab, already active.
- Have the Google Sheet open in a third tab.
- Have a Gmail inbox open for the notification.
- Use `scripts/sample_invoice_payload.json` to know the expected output before going live.

---

## Demo Sequence

### Step 1 — Frame the problem *(~30 seconds)*

> "Finance teams receive invoices from dozens of suppliers in different formats. Someone has to open each one, find the vendor, invoice number, dates, and total, then key it all into a spreadsheet. That's slow, inconsistent, and easy to get wrong. This workflow eliminates that manual step."

### Step 2 — Show the architecture *(~30 seconds)*

Point to `docs/architecture.md` or draw attention to the n8n canvas.

> "The flow is: invoice lands in Google Drive, n8n picks it up, OCR.Space reads the document, OpenRouter structures the fields, we validate the result, log it in Google Sheets, and send a notification. Nine nodes, one clear path."

### Step 3 — Upload the invoice *(~30 seconds)*

Drag the sample invoice PDF into the `/Incoming Invoices` Google Drive folder.

> "I'm uploading a Philippine peso invoice from NorthStar Office Solutions. The trigger is polling every minute, so n8n will fire shortly."

### Step 4 — Show the workflow trigger *(~30 seconds)*

Switch to n8n. Show the execution starting.

> "The Google Drive Trigger fired. It captured the file ID, name, MIME type, and created time. The workflow is now downloading the file to send to OCR."

### Step 5 — Explain the OCR and AI steps *(~60 seconds)*

Click into the `OCR.Space Extract` node output.

> "OCR.Space reads the raw text from the PDF. That text is unstructured — column headers and line items are mixed together. That's where OpenRouter comes in."

Click into the `OpenRouter Extract Fields` node output. Show the JSON.

> "I'm passing the OCR text to OpenRouter — which routes to the best available model — with a constrained prompt. The prompt tells the model to return only a JSON object — no explanations, no guessing. If a field is missing, it returns null. Temperature zero keeps the output stable."

Click into the `Parse Structured Invoice` node.

> "This node validates the result. It checks vendor name, invoice number, and total amount. If any are missing or the JSON is malformed, the invoice goes to the manual review branch instead of being logged as clean data."

### Step 6 — Show Google Sheets *(~30 seconds)*

Switch to the Google Sheet tab. Show the new row.

> "The invoice passed validation. Here it is in the sheet — vendor NorthStar Office Solutions, invoice INV-2026-0412, total PHP 16,542.40, status Ready for Review. All 15 columns populated automatically."

### Step 7 — Show the Gmail notification *(~20 seconds)*

Switch to Gmail.

> "And here's the success notification. Vendor, invoice number, total, and file name in one email. Stakeholders know it was processed cleanly without opening n8n."

### Step 8 — Close with controls *(~30 seconds)*

> "If the OCR text is empty, or the AI output is missing required fields, or confidence notes flag uncertainty — the workflow routes to the manual review path. It still writes a row to the sheet, marks it as Manual Review, and sends a separate alert email with the failure reason. Finance always has visibility. Nothing fails silently."

---

## Strong Closing Line

> "This design prioritizes speed of delivery, explainability, and practical business value while leaving room for stronger controls in a production version — things like deduplication, a database backend, retry logic, or a Slack integration."

---

## Backup: If Live Demo Fails

Reference `scripts/sample_invoice_payload.json` and walk through the expected output manually:

- Vendor: `NORTHSTAR OFFICE SOLUTIONS`
- Invoice Number: `INV-2026-0412`
- Invoice Date: `2026-04-01`
- Due Date: `2026-04-15`
- Currency: `PHP`
- Subtotal: `14770.00`
- Tax: `1772.40`
- Total Amount: `16542.40`
- Suggested Category: `Office Supplies`
