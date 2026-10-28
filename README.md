# AI Lead Qualification & CRM Automation (n8n)

A portfolio demonstration of a lead intake automation built with **n8n**, **OpenAI Responses API**, **Google Sheets**, and **Gmail**.

## What it does

1. Accepts a lead through an HTTP POST webhook.
2. Validates required fields and rejects invalid submissions.
3. Uses structured AI output to classify the request and set its priority.
4. Creates/updates a CRM row in Google Sheets using `submission_id`.
5. Checks `notification_sent_at` before notifying about a high-priority lead.
6. Sends an email and writes its timestamp after a successful send.
7. Includes branches for selected service-error responses.

## Workflow

`Webhook → Validate → OpenAI → Parse → Lookup existing lead → Preserve notification state → Upsert to Sheets → Notification decision → Gmail (when due) → Save timestamp → Respond`

## Demo status

Local integration tests have been performed with sample leads, including a low-budget lead and repeat-submission scenarios. The workflow is a **portfolio demo**, not a production-hardened service. Failure paths and simultaneous duplicate requests are not fully guaranteed; Sheets is not a transactional queue. A robust deployment would add secure authentication, rate limiting, durable retries/queueing, alerting, and concurrent-request idempotency.

## Setup (self-hosted n8n)

1. Import `workflow.template.json` into n8n.
2. Configure **OpenAI HTTP Bearer authentication** in the OpenAI node; do not paste secrets into the JSON file.
3. Create a Google Spreadsheet with a sheet named `Leads` and headers from `crm_headers.csv`.
4. Set the Google Sheets document and sheet selections for **each** Google Sheets node, and reconnect Google Sheets OAuth credentials.
5. Configure Gmail OAuth and change the destination from `manager@example.com` to the recipient you control.
6. Check the Webhook **Test URL** in your imported workflow; use the URL provided by your n8n instance.
7. Save, click **Execute Workflow**, and send a test POST. Test URLs only listen for the next request; for a continuous deployment, activate/publish the workflow and use its production webhook URL.

### Windows PowerShell test

```powershell
$body = @{
  submission_id = "demo-001"
  name = "Example Client"
  email = "client@example.com"
  company = "Example Co"
  budget = 7500
  message = "We need an AI chatbot integrated with our CRM."
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:5678/webhook-test/ai-lead-qualification-demo" -Method POST -ContentType "application/json" -Body $body
```

## Security note

This repository is a **sanitized template**. Exported credentials, the original recipient address, spreadsheet IDs, instance IDs, and other account-specific identifiers were removed. Never commit API keys, OAuth secrets, client datasets, or unredacted execution logs. A previously shared OpenAI API key should be revoked and replaced if not already rotated.

## Portfolio talking points

- Webhook integrations and payload validation
- Structured outputs and rule-based safety constraints around AI decisions
- Google Sheets CRM upsert / basic duplicate handling
- High-priority email alerts with notification-state tracking
- API-oriented error handling

**Stack:** n8n · OpenAI API · Google Sheets API · Gmail API · JavaScript · Docker
