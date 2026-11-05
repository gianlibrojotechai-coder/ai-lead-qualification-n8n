# Known limitations and remaining verification gaps

This workflow is an inactive, unpublished portfolio demo built and tested with synthetic data. The points below are not caveats added for form; each one was observed or deliberately left unverified during the project.

## Limitations of the design

- Semantic variability. The same lead text produced different criterion scores across model calls (Fit 2, then 0, then 2 for one input) before and after prompt clarifications. Temperature and reasoning settings reduce but do not remove this. The eleven mechanical rules prove format, allowed values, and arithmetic; they do not prove that a lead was judged correctly. Semantic review is a human step.
- Non-atomic lookup and update. The exact-one-match check and the row update are separate calls. A duplicate row appended between them is not prevented; the design only refuses to update when it sees more than one match.
- No deduplication. Each submission is a separate record by design. Re-running a Hot lead creates a second row and, if it qualifies Hot again, a second email.
- Zero retries. Transient failures end the execution and require manual investigation. Uncertain writes and sends are reported as UNCONFIRMED and are never resolved automatically.
- Provider acceptance is not delivery. A Gmail message id means the provider accepted the message. Nothing in the workflow establishes delivery or reading, and absence from an inbox never proves non-delivery.
- Free-text interpretation. Timeline and budget are interpreted by the model against rubric anchors, not parsed.
- Client-side form check. The browser enforces required fields; a direct trigger execution bypasses that, which the workflow's own check catches.
- Plain-text notification. The company name is bounded only in the subject; the body carries the stored text as is.
- Synthetic data only. No real lead has ever passed through the workflow, and the privacy posture of the prompt (only the business need, timeline, budget, and submission date are sent to the model) has not been reviewed for real data.

## Remaining verification gaps

- Provider error paths (rate limit, outage, truncation) have no live evidence; the failure classifier was tested locally and with pinned simulations only.
- Google Sheets lookup errors and update errors have no live evidence; classification rests on local tests.
- Gmail send errors have no live evidence; the unconfirmed path was simulated.
- Warm routing has simulation evidence only; the live Warm qualification predates the routing nodes.
- The state of a row after an unconfirmed update has never been observed; a recovery procedure is defined but has never been exercised.
- Instance retention and pruning settings were not visible through the tooling used; executions from the first day were still readable two days later, which is all that was observed.
- Whether execution details expose more than intended (they contain the full lead text, the raw model reply, and encrypted runtime data) is a judgement that was not made.
- The workflow was never activated. Behaviour in production mode, including how execution data is saved and how the form behaves at its production URL, is untested.
- The original n8n version (2.32.5) is user-reported. Importing the sanitized export into another instance was not tested; the structural checks on the export were local only.
- Two cosmetic validation warnings exist on the development copy after an editor save collapsed two default-valued parameters. They did not change behaviour; the export writes those values explicitly.

## What would close the gaps

A separately approved set of live failure injections (an invalid model name for a provider rejection, a revoked sheet permission for a lookup error, a temporarily invalid recipient for a send rejection), one live Warm submission, an import test on a clean instance, and a decision on execution-detail exposure. None of these were authorized as part of the project and none are required for the demo as documented.
