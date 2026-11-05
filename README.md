# AI Lead Qualification & Scoring Automation (n8n + OpenAI + Google Sheets)

A portfolio demo in n8n: a lead submits a web form, the record is prepared and stored in Google Sheets, a language model scores it against an explicit rubric, eleven mechanical rules validate the answer, the row is updated, and hot leads trigger one notification email. Every failure stops with a short, safe diagnostic instead of failing silently.

The business is fictional and all data is synthetic.

## Start here

- `PROJECT-EXPLANATION.md`: the problem, the 42-node architecture in seven stages, the rubric, and the design decisions.
- `WALKTHROUGH.md` and `walkthrough-rendered.mp4`: a guided tour of the workflow.
- `EVIDENCE-SUMMARY.md`: every claim with its evidence, labelled live, simulated or local.
- `LIMITATIONS.md`: what is not proven or not built.
- `IMPORT.md`: how to import and configure it (placeholders such as `<<SPREADSHEET_ID>>` replace all private values).
- `workflow-sanitized.json`: the n8n workflow export, no credentials.

## Design rules

- The model only scores; code recomputes totals and the class from the criterion scores and rejects anything inconsistent.
- Submitted text is placed in delimited data blocks and treated as data, not instructions.
- Storage is create-then-update keyed by a UUID, and a row is updated only when exactly one match exists.
- External calls have retries off, and every failure ends visibly with a bounded diagnostic that never contains the submitted text.
