# Local walkthrough: narration and shot list

Status: this is the narration script and shot list. A rendered presentation of it exists as walkthrough-rendered.mp4 in this folder (1280x720, about 4 minutes 51 seconds): eight text slides drawn from this file and the stage diagram, with the narration read by Windows text-to-speech. It is not screen footage; it shows no editor, execution, inbox, or spreadsheet. The tooling used here cannot capture screen or audio, so a human-presented screen recording, if ever wanted, is still to be made. Any such recording must show only the files in this folder; it must not show the private n8n editor, any inbox, the project work log, account details, or the unsanitized spreadsheet. Nothing shown may be presented as execution footage: every visual below is a file, a diagram drawn from the sanitized export, or a table quoted from EVIDENCE-SUMMARY.md.

Target length: about six minutes. Evidence aliases (RUN-…, SIM-…) are defined in EVIDENCE-SUMMARY.md. "Recorded result" below always means a result recorded during development and summarized in that file, not a result produced during the walkthrough.

## Shot 1: the problem (about 45 seconds)

On screen: PROJECT-EXPLANATION.md, sections "The problem" and "Architecture".

Narration: "This is an inactive, unpublished portfolio demo built in n8n. A fictional automation consultancy wants every inbound lead recorded, scored the same way against a written rubric, surfaced when it is hot, and loud when something fails. The workflow runs in seven stages: intake, preparation, storage, qualification by a language model, a validated write-back to the same spreadsheet row, routing, and a failure model that ends a run visibly instead of silently. Everything you will see was exercised with synthetic leads only."

## Shot 2: the sanitized workflow structure (about 75 seconds)

On screen: workflow-sanitized.json opened in a text editor, scrolled to the node list; then the stage diagram below (draw it as a slide or read it from this file; do not show the live editor).

```
Lead Intake Form
  -> Prepare Lead -> Required Fields Present? -> Submission Timestamp Valid?
  -> Generate Lead ID -> Build Lead Record
  -> Store Lead Row -> Storage Confirmed?
  -> Build AI Input -> Qualify Lead (+ OpenAI Chat Model) -> Validate AI Response | Mark AI Call Failed
  -> Carry Outcome -> Find Lead Row -> Count Matches -> Route Match Count (0 / 1 / many)
  -> Route Outcome -> Update Row: Qualified | Update Row: Qualification Failed -> Update Confirmed?
  -> Outcome Qualified? -> Route Classification (Hot / Warm / Cold / unsupported)
  -> Hot: Notification Config -> Build Hot Notification -> Send Hot Notification -> Notification Confirmed?
  -> Warm: End: Warm      Cold: End: Cold
Fail nodes (eleven Stop and Error nodes) hang off every check and every external call.
```

Narration: "The export has forty-two nodes: forty-one on the main canvas and one model sub-node. Three placeholders replace the spreadsheet id, the tab id, and the notification recipient; no credentials or webhook identifiers are included. Notice the shape: every external call has an error output wired to a fail node, no node retries, and the two silent endings for Warm and Cold are visible nodes rather than dangling outputs."

## Shot 3: the rubric and the validator (about 60 seconds)

On screen: the Qualify Lead system message inside the JSON, then the Validate AI Response code.

Narration: "The system prompt carries the whole rubric: Fit zero, two or four; Clarity zero to three; Timeline zero to two; Budget zero or one; a gate that zeroes everything when Fit is zero; Hot seven to ten, Warm four to six, Cold zero to three. The lead's text is wrapped as data with an instruction that it is not instructions. The model answers with one JSON object. Then n8n applies eleven rules in order and recomputes the total and class itself. The model's class is a claim; the arithmetic is the authority."

## Shot 4: recorded Hot and Cold results (about 60 seconds)

On screen: EVIDENCE-SUMMARY.md, the "Paid model calls" table.

Narration: "These are recorded results, each a real submission through the real services. RUN-HOT: a dental practice with a clear repetitive process, a one-month timeline and a stated budget scored four, three, two, one, ten, Hot. RUN-FITGATE: an urgent, funded request for a lawyer scored Fit zero, the other criteria not scored, Cold, because the gate rests on scope, not eagerness. RUN-ADVERSARIAL: a business need that told the model to return a Hot ten was scored on its actual request, a website redesign, as Cold. RUN-HOT-NOTIFY went end to end: qualified Hot, row updated, one plain-text email accepted by the provider. The provider's message id proves acceptance; the owner's report that it arrived is a separate, user-reported fact."

## Shot 5: validation failures, simulated (about 45 seconds)

On screen: EVIDENCE-SUMMARY.md, the "Coverage by area" table, rows for qualification and row matching.

Narration: "The failure paths were exercised with simulations: pinned inputs run on the real n8n instance with the model and the sheet bypassed. SIM-V9 fed a reply whose class contradicted its total; the row was marked qualification_failed with code V9 first, and only then was the run ended as failed. SIM-MATCH-MULTI fed two rows with the same lead id and the update was refused. Each simulation establishes only the downstream logic it exercised: SIM-V9 shows that a reply failing rule V9 is rejected, written to the row as qualification_failed with its code, and then ended as a failed run; SIM-MATCH-MULTI shows that a duplicate lead id is detected and the update is refused. Neither verifies the bypassed model call or the bypassed Google Sheets operations; the summary labels each row accordingly."

## Shot 6: the L3 regression, honestly (about 60 seconds)

On screen: EVIDENCE-SUMMARY.md, "The L3 history in one place".

Narration: "One vague lead exposed the limits of mechanical checks. RUN-COLD-FIRST scored it exactly as the reference expected but paraphrased a wish as a pain point, which the rules say must be null. A one-sentence prompt clarification fixed that, and RUN-L3-RETEST-1 then under-scored Fit, zero instead of two. A second one-sentence clarification stated that Fit and pain point are independent. RUN-L3-REGRESSION produced two, one, zero, zero, Cold, pain point null: the reference result. That is one passing run, recorded as one passing run. The explanation for the earlier slip is a hypothesis. All three rows stay in the sheet unedited."

## Shot 7: limitations and status (about 45 seconds)

On screen: LIMITATIONS.md, the two headed lists.

Narration: "What this demo does not claim: it is not production ready; the model's scores vary between runs; the lookup and the update are two calls, not a transaction; nothing deduplicates, so a manual rerun can send a second email; a message id is acceptance, not delivery; and the error branches for the model, the sheet, and the mailer were verified by simulation and local tests only. The workflow is inactive and unpublished, and that is how it stays until someone decides otherwise, with the costs and side effects written down first."

## Production notes for the person recording

- Show only: workflow-sanitized.json, PROJECT-EXPLANATION.md, EVIDENCE-SUMMARY.md, LIMITATIONS.md, and a diagram drawn from the structure in Shot 2.
- Never show: the n8n editor, the executions list, any inbox, the spreadsheet, the project work log, credential pages, or account settings.
- Do not run anything. A live run would cost a model call and may send an email; it is not part of this walkthrough.
- After recording, review the footage for any private identifier (addresses, ids, URLs) and for any sentence that states more than EVIDENCE-SUMMARY.md supports.
