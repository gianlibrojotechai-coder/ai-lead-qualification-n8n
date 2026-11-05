# Import and configuration guide

This folder contains a sanitized export of the P001 lead qualification workflow, an inactive, unpublished portfolio demo. It is not production ready. Importing it gives you the structure, prompts, validation, and error handling; you supply your own credentials, spreadsheet, and recipient.

## What is in the export

`workflow-sanitized.json` holds 42 canvas nodes: 41 main nodes plus one AI model sub-node. Credentials, credential identifiers, webhook identifiers, the original spreadsheet identifiers, the original recipient address, and all instance metadata have been removed. Three placeholders mark the values you must supply. Two node parameters that the original editor had collapsed to their defaults are written out explicitly in this export so the import is unambiguous: the OpenAI Chat Model's "Use Responses API" flag (true) and the Gmail node's resource and operation (message, send). They do not change behaviour.

The two helper workflows used during development (a run-once sheet setup and a read-only read-back) are not exported. Their jobs are covered by the manual steps below.

## Prerequisites

- An n8n instance. The original was reported by its owner as version 2.32.5; that figure is user-reported, not independently verified. Node versions in the export: Form Trigger 2.6, Code 2, If 2.3, Switch 3.4, Set 3.4, Crypto 2, Google Sheets 4.7, Gmail 2.2, Basic LLM Chain 1.9, OpenAI Chat Model 1.3, Stop and Error 1, No Operation 1.
- A Google Sheets OAuth2 credential with edit access to the spreadsheet you create.
- A Gmail OAuth2 credential for the mailbox that will send the Hot notification.
- An OpenAI API credential. The tested model is gpt-5.4-mini. Inference is paid; every qualification run makes one model call.

## Placeholders and where they appear

| Placeholder | Meaning | Nodes |
|---|---|---|
| `<<SPREADSHEET_ID>>` | The id of your Google spreadsheet (the long token in its URL) | Store Lead Row, Find Lead Row, Update Row: Qualified, Update Row: Qualification Failed (parameter `documentId.value`) |
| `<<LEADS_SHEET_GID>>` | The numeric gid of the tab named Leads | the same four nodes (parameter `sheetName.value`) |
| `<<DEMO_RECIPIENT_EMAIL>>` | The mailbox that receives Hot notifications; must be an address you control, never a lead's address | Notification Config (assignment `recipient_email`) |

## Steps

1. Create the spreadsheet. One spreadsheet, one tab named `Leads`, and this header row in row 1, in this exact order:
   `lead_id, received_at, status, failure_reason, company_name, contact_email, business_need, timeline, contact_name, budget, fit_score, clarity_score, timeline_score, budget_score, lead_score, classification, summary, pain_point, recommended_action`.
   Note the spreadsheet id from the URL and the gid of the Leads tab (visible in the URL as `gid=`; a first tab is often 0).
2. Import `workflow-sanitized.json` in n8n (Workflows, Import from File). Leave the workflow inactive.
3. Replace the placeholders. Open each of the four Google Sheets nodes and set the document id and the sheet gid; open Notification Config and set the recipient.
4. Attach credentials. Google Sheets credential on the four Sheets nodes; Gmail credential on Send Hot Notification; OpenAI credential on OpenAI Chat Model. The export carries no credentials, so n8n will show those nodes as needing one.
5. Before any test, prevent sending: either disable the Send Hot Notification node (right-click, Deactivate) or leave the recipient placeholder unresolved so the send cannot address anyone. Re-enable only when you intend to send to the configured mailbox.
6. Test with the editor's Test workflow button and the Form Trigger's Test URL. Use synthetic data only.

## Webhook identifiers

The export deliberately omits the Form Trigger's webhook identifier and the Gmail node's internal webhook identifier. In our experience n8n generates a new identifier when a workflow without one is imported, but this was not tested as part of the project; verify in your instance by opening the Form Trigger after import and reading its Test URL. If no URL appears, add a value in the node's "Path" option or re-create the trigger node.

## Costs and side effects during testing

- Each qualification run calls the OpenAI model once. With gpt-5.4-mini the project estimated roughly one third of a US cent per call from token counts and the published price list; the billed amount for the project is unknown.
- A Hot classification sends one real email to the configured recipient. Disable the send node for initial configuration.
- A single row is appended to the Leads tab for every valid submission and then updated once with the qualification result.

## Provider and model substitutions

Only gpt-5.4-mini through the OpenAI Chat Model sub-node was tested. Any other provider or model is an untested alternative, not a guaranteed drop-in replacement: it needs the sub-node swapped, retries disabled at the node level if the node exposes them, JSON output requested, and the full validation and regression checks repeated. The validation rules in the workflow do not depend on the provider.

## Environment notes

- The Code node sandbox on the original instance exposed neither a global `crypto` object nor `require('crypto')`, so lead ids are generated by the native Crypto node. Keep that node even if your sandbox allows more.
- The Google Sheets read used for row matching sets "Always Output Data" so that a zero-match read still reaches the match-count guard. Do not remove that setting.
- All retry settings are off by design; failures end the execution with a bounded diagnostic message.
