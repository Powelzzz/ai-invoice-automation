# Interview Q&A

## Why did you choose this problem?

I chose invoice automation because it is a practical business problem with clear operational value. Invoice handling is repetitive, error-prone, and common across many organizations, so it is a strong use case for workflow automation and AI-assisted data extraction.

It also makes a good technical assessment because it combines document processing, system integration, validation, exception handling, and business impact in one solution.

## Why did you choose `n8n`?

I chose `n8n` because it is well suited for fast, API-driven automation. It provides a clear visual workflow, supports the required integrations, and makes the orchestration easy to explain during a demo.

For an assessment, `n8n` is a strong fit because it shows practical solution design without spending unnecessary time on building orchestration code from scratch.

## Why use OCR plus AI instead of OCR alone?

OCR and AI solve different parts of the problem.

OCR converts the invoice file into raw text, but that text is often unstructured and inconsistent. Supplier invoices vary in layout, wording, and formatting, so OCR alone does not reliably produce clean business fields.

AI is used after OCR to interpret the raw text and map it into structured fields such as vendor name, invoice number, dates, currency, and totals. This combination is more flexible and accurate than relying only on OCR or only on fixed parsing rules.

## How do you prevent hallucinations?

I reduce hallucination risk by limiting the model's job and validating its output.

Key controls include:

- A constrained prompt that asks for JSON only
- Instructions not to invent missing values
- Returning `null` when information is not present
- Low-temperature model settings for more predictable output
- Schema checks before downstream use
- Exception handling when required fields are missing or malformed

The model is treated as a parsing assistant, not a source of truth.

## What validation logic did you apply?

I added validation between the AI extraction step and the final logging step.

The workflow checks that:

- The output is valid JSON
- Required fields such as `invoice_number` and `total` are present
- Amount fields are numeric
- Dates are in the expected format when available
- Empty OCR responses are rejected early

If validation fails, the workflow does not continue as if the invoice were clean. It moves the item to an exception or manual review path.

## How is the solution cost-efficient?

The design is cost-efficient because it uses lightweight tools for each step and keeps the AI usage focused.

Reasons it stays efficient:

- `n8n` reduces custom development overhead
- OCR is only used when a new file arrives
- OpenRouter is only used after OCR, for structured extraction
- Google Sheets is sufficient for demo-scale tracking
- Gmail handles notifications without extra infrastructure

This keeps the architecture simple and affordable while still demonstrating business value.

## How would you scale this solution?

For higher volume, I would strengthen the architecture in stages rather than redesign it all at once.

Typical improvements would include:

- Replacing Google Sheets with a database or accounting system integration
- Adding a queue for burst handling and retry control
- Introducing deduplication and idempotency checks
- Storing confidence scores and review outcomes
- Adding monitoring, logging, and alerting
- Splitting OCR, extraction, validation, and posting into clearer service boundaries if needed

The current version is intentionally optimized for assessment clarity, but the design can evolve into a more robust production workflow.

## How did you think about security?

Security starts with limiting exposure of invoice data and protecting credentials.

The main controls are:

- Store API keys outside source control
- Use `n8n` credentials or environment variables for secrets
- Restrict access to Google Drive, Google Sheets, and Gmail resources
- Avoid storing more raw document data than necessary
- Log operational events without exposing sensitive invoice content where possible

For production, I would also add stronger access controls, audit logging, retention policies, and data classification rules.

## What happens when the automation is not confident or the output is incomplete?

The workflow should not force uncertain results into the final record.

If important fields are missing, if OCR text is unreadable, or if the AI response fails validation, the invoice moves to a manual review path. That path records the failure reason, preserves the source reference, and alerts the responsible team by email.

This approach is important because finance workflows need controlled exceptions, not silent failures or guessed data.

## Why is a manual review path important?

A manual review path is necessary because invoice automation should improve operations without weakening controls.

Even a strong workflow can receive low-quality scans, unusual invoice layouts, or incomplete documents. The manual review step ensures those cases are visible, traceable, and handled safely by a person instead of being posted incorrectly.

In other words, the goal is not full automation at any cost. The goal is reliable automation with a safe fallback path.
