# Evidence summary

All evidence comes from the project's development instance between 2026-10-15 and 2026-10-17. Executions, sheet rows, and the email are referred to by aliases; the private mapping from aliases to execution ids, row numbers, and message ids is kept in the project's work note, not in this folder.

## Labels

- LIVE: a real submission through the real services (form or connector trigger, real Google Sheets writes, real model call, real Gmail send where reached).
- SIM: simulated test inputs executed on the real n8n instance with the trigger and every provider or service node pinned to supplied data. The n8n node logic ran; provider and service behaviour was bypassed. A simulation never establishes live provider or service behaviour.
- LOCAL: the identical function run in a local Node.js process, outside n8n.
- USER: observed and reported by the project owner (a screenshot or a sentence), not by tooling.
- NONE: no evidence.

## Paid model calls (ten in total, all gpt-5.4-mini, all with retries off)

| Alias | Input | Result | Expected | Match |
|---|---|---|---|---|
| RUN-HOT | reference example 1 | 4/3/2/1 = 10 Hot | same | exact |
| RUN-WARM | reference example 2 | 4/2/0/0 = 6 Warm | same | exact |
| RUN-COLD-FIRST | reference example 3 | 2/1/0/0 = 3 Cold, pain point filled | pain point null | scores exact; pain point wrong |
| RUN-HOT-NOBUDGET | reference example 4 | 4/3/2/0 = 9 Hot | same | exact |
| RUN-FITGATE | reference example 5 | 0/gate = 0 Cold | same | exact |
| RUN-ADVERSARIAL | injected instructions | 0/gate = 0 Cold, injection ignored | no override | pass |
| RUN-BROWSER-HOT | browser submission | 4/3/2/1 = 10 Hot | Hot | pass |
| RUN-L3-RETEST-1 | example 3 after first clarification | 0/gate = 0 Cold, pain point null | 2/1/0/0 | pain point fixed; Fit wrong |
| RUN-HOT-NOTIFY | browser submission, routing live | 4/3/2/1 = 10 Hot, email sent | Hot + email | pass |
| RUN-L3-REGRESSION | example 3 after second clarification | 2/1/0/0 = 3 Cold, pain point null | same | exact (one run) |

Every one of the ten responses passed the eleven mechanical rules on the first attempt. No provider error, quota error, or truncation occurred, so none of those live behaviours is evidenced.

Cost: the chain node reported about 900 input and 160 output tokens for one call, flagged as an estimate, with reasoning tokens not itemized. At the published prices for gpt-5.4-mini on 2026-10-16 the project estimated roughly one third of a US cent per call and about three US cents for all ten. These are estimates from token counts; the billed total is unknown because the billing account was not inspected.

## Email (one in total)

RUN-HOT-NOTIFY produced one plain-text message to the configured demo mailbox. LIVE evidence: the Gmail node returned a message id, which establishes provider acceptance. USER evidence: the owner reported that the email arrived. Delivery is not claimed by the workflow.

## Coverage by area

| Area | Behaviour | Evidence |
|---|---|---|
| Input validation | required field absent, blank, or whitespace, all names listed in order | LIVE (three runs), SIM (non-string value) |
| Input validation | optional non-string or whitespace becomes null | LIVE, SIM |
| Input validation | timestamp absent or without offset stops the run before any write | SIM; LIVE (RUN-TIMESTAMP-GUARD, two editor runs whose pinned trigger lacked the timestamp) |
| Input validation | browser blocks a missing required field | USER screenshot |
| Input validation | browser +02:00 timestamp converted to UTC | LIVE (four browser runs) |
| Storage | append, echo check, literal text, nulls as blank cells, numeric zero numeric | LIVE (sixteen appends, five read-backs) |
| Storage | append rejected before write (missing tab) | LIVE (two runs) |
| Storage | other append errors, mismatched echo | LOCAL only; NONE live |
| Qualification | eleven validation rules and class boundaries | LOCAL (all rules); SIM (V1, V9) |
| Qualification | live valid responses | LIVE (ten calls above) |
| Qualification | model-call failure stored then execution failed | SIM; LOCAL |
| Qualification | real provider error, quota, truncation | NONE |
| Qualification | embedded instructions ignored | LIVE (RUN-ADVERSARIAL) |
| Row matching | zero matches stops without update | SIM; LIVE read-only probe of a zero-item read |
| Row matching | duplicate matches stop without update | SIM |
| Row matching | lookup error diagnostic | LOCAL reasoning only; NONE executed |
| Result update | qualified update with gated blanks and numeric zeros | LIVE |
| Result update | failed-status update then execution failed | SIM |
| Result update | update error classifier | LOCAL only |
| Result update | mismatched update echo | SIM |
| Routing | Hot to notification | LIVE (RUN-HOT-NOTIFY); SIM |
| Routing | Warm ends silently | SIM only |
| Routing | Cold ends silently | LIVE (RUN-L3-REGRESSION); SIM |
| Routing | unsupported class stops with fixed diagnostic | SIM |
| Notification | provider accepted the send | LIVE (one) |
| Notification | recipient receipt | USER (one) |
| Notification | missing message id, send error classifier, subject sanitization | SIM; LOCAL; SIM |
| Saved evidence | failed executions listed as failed with node data | LIVE (every failure run) |

## The L3 history in one place

RUN-COLD-FIRST (scores right, pain point wrong) led to a one-sentence prompt clarification on the pain point rule. RUN-L3-RETEST-1 (pain point right, Fit wrong) led to a second one-sentence clarification stating that Fit and pain point are independent. RUN-L3-REGRESSION then matched the reference exactly. All three rows remain in the sheet unedited. The explanation that the first clarification biased Fit is a hypothesis; one passing run demonstrates that the corrected prompt can produce the required result, not that it always will.
