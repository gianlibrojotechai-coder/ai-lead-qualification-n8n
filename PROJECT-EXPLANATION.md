# AI-assisted lead intake and qualification in n8n

Privacy: PUBLIC-grade material (sanitized, aliases only, no private identifiers). It stays local unless the owner chooses to share or publish it.

An inactive, unpublished portfolio demo. It takes a lead from a web form, prepares and stores it, asks a language model to score it against an explicit rubric, validates the answer mechanically, writes the result back to the same spreadsheet row, and notifies a person for Hot leads. Everything in it was exercised with synthetic data only. It is not production ready.

## The problem

A small, fictional AI automation consultancy wants incoming inquiries handled consistently: every lead recorded, each one scored the same way, hot ones surfaced promptly, and failures visible rather than silent. The demo business scope is deliberately narrow (repetitive rule-based inquiry handling, data entry between tools, document and email processing, reminders, simple reporting) so that fit can be judged.

## Architecture (42 nodes in one workflow, seven stages)

1. Intake. An n8n Form Trigger with six fields: company_name, contact_email, business_need, timeline (required), contact_name, budget (optional). The page acknowledges with a neutral sentence that promises nothing about storage, qualification, or notification.
2. Preparation. A Code node trims every field, turns absent, blank, or non-string optional values into null, lists missing required fields, and converts the form's submission timestamp (which carries an explicit offset) to UTC. Two guards end the run with a diagnostic if a required field is missing or the timestamp is unusable. A native Crypto node generates a UUID v4 lead id. The prepared record has exactly ten fields.
3. Storage. The record is appended to a Google Sheet with 19 columns (ten intake and tracking columns, nine reserved for results). Values are written literally, so a company name like `=1+1` stays text. The append response is checked for the same lead id before the run continues.
4. Qualification. The business need, timeline, budget, and submission date are placed in delimited data blocks under a system prompt that holds the full rubric and tells the model the blocks are data, not instructions. The model answers with one JSON object. A Code node then applies eleven validation rules in order: parse, exact key set, types, allowed values, the Fit gate, the arithmetic total, class consistency with the recomputed total, non-empty prose, and a placeholder check on the pain point. The rubric arithmetic is authoritative; the model's own total and class are claims that are verified, not trusted.
5. Row update. The workflow looks the row up by lead id, requires exactly one match, and only then updates it: either to qualified with the four criterion scores and five outputs, or to qualification_failed with a failure code. A failed qualification is stored first and then the execution is deliberately ended as failed, so the sheet and the execution list agree.
6. Routing. On a confirmed qualified update, the stored classification selects the branch: Hot sends one plain-text email to a configured recipient; Warm and Cold end at visible terminal nodes; anything else stops with a fixed diagnostic. Routing never changes the sheet.
7. Failure model. Every external operation has retries off. A detected failure ends the execution as failed with a bounded diagnostic of the form `CODE lead_id=… step=… detail=…` that never carries submitted text or raw provider messages. Codes distinguish a rejection known to have happened before any write (FAILED) from an outcome that cannot be known (UNCONFIRMED). Recovery is investigation only: read the sheet back by lead id, check the sending mailbox, record what is known, and never retry automatically.

## The rubric

Four criteria, integer points, total 0 to 10. Fit (0, 2, or 4) from the business need against the demo scope; Clarity (0 to 3) from the business need; Timeline (0 to 2) from the timeline anchored on the submission date; Budget (0 or 1) from the budget text. If Fit is 0 the other three are not scored (null), the total is 0, and the class is Cold. Otherwise the total is the sum and the class is Hot (7 to 10), Warm (4 to 6), or Cold (0 to 3). A missing budget costs nothing beyond its own point; an urgent, funded, out-of-scope request still scores 0. Fit and pain point are judged independently: automation interest without a described problem earns Fit 2 while the pain point stays null.

## What the validation proves and what it does not

The eleven rules prove format, allowed values, arithmetic, and internal consistency. They do not prove that the model judged a lead correctly. Semantic review against the rubric was done by a person on each live run, and one case had to be corrected twice (see the evidence summary and limitations): the first live run paraphrased a wish as a pain point, the re-test fixed that but under-scored Fit, and a third run after a second prompt clarification produced the reference result. That history is part of the portfolio, not hidden from it.

## Storage design

Create-then-update, keyed by a UUID rather than the email address, so a repeat inquiry from the same address stays a separate record. Null becomes an empty cell and numeric zero stays numeric, so a blank result cell always means "no accepted result" or "not scored", never "scored zero". The lookup and the update are two calls; the exact-one-match check happens immediately before the update but is not an atomic transaction.

## Routing and notification boundaries

The notification is one send attempt per execution. A Gmail message id proves that the provider accepted the message, not that it was delivered or read. Leads are never contacted; the recipient comes from configuration only.

## Evidence and labels

Every claim in this folder is backed by recorded evidence that is labelled as live (a real submission through real services), simulated (pinned inputs executed on the real n8n instance with the provider or service node bypassed), local (the same function run outside n8n), or user-reported (observed by the owner, not by tooling). See EVIDENCE-SUMMARY.md and LIMITATIONS.md.
