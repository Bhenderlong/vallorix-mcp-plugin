# Working with the Vallorix MCP

Guidance for any agent using the Vallorix MCP tools, regardless of which skill started the conversation.

## Model of the data
- **Framework** (SOC 2, ISO 27001, HIPAA, CMMC, …) → made up of **Controls**.
- **Control** → satisfied by one or more **Measures** (the implementation items). A measure can satisfy controls across *several* frameworks (`listMeasureControls`).
- **Measure** has a `state`: `IMPLEMENTED`, `NOT_APPLICABLE`, `NOT_STARTED`, `IN_PROGRESS`, `NOT_IMPLEMENTED`, `UNKNOWN`. "Open gap" = anything not `IMPLEMENTED`/`NOT_APPLICABLE`.
- **Evidence** (`listMeasureEvidences`) proves a measure, with an automation state: `FULLY`, `PARTIALLY`, `AUTOMATION_READY`, `MANUAL`, plus validity/expiry.
- **Finding** is an open nonconformity/observation with `status` (`OPEN`/`IN_PROGRESS`/`CLOSED`/`RISK_ACCEPTED`/`MITIGATED`/`FALSE_POSITIVE`), `priority` (HIGH/MEDIUM/LOW), and `kind` (MAJOR/MINOR_NONCONFORMITY/OBSERVATION/EXCEPTION).

## Principles
1. **Ground everything in the real requirement.** Load `getMeasure`/`getFinding` + `listMeasureControls` before advising. Never remediate from general knowledge alone.
2. **Never weaken security** to make a control pass. Flag it instead.
3. **Minimal change.** Satisfy the control, nothing more.
4. **Never dead-end.** If an ID/filter returns nothing, fall back to `listMeasures`/`listFindings` and fuzzy-match.
5. **Confirm before writes.** `updateMeasure`, `updateFinding`, `addFinding`, etc. change the user's compliance record — show what you'll change and get a yes.
6. **Respect scope.** The user's Vallorix role bounds what's visible/editable; don't assume more.
7. **Sovereign by default.** This talks only to the user's Vallorix instance (`VALLORIX_MCP_URL`). Say so when it matters — data doesn't leave their boundary.
8. **Scannable output.** Tables for lists, code blocks for diffs, bold for the control a fix satisfies.

## Common tools
`listFrameworks` · `getFramework` · `listControls` · `getControl` · `listControlMeasures` · `listMeasures` · `getMeasure` · `updateMeasure` · `listMeasureControls` · `listMeasureEvidences` · `listMeasureTasks` · `listFindings` · `getFinding` · `updateFinding` · `listRisks`. (The server exposes ~285 tools across the full GRC model — browse them if a task needs more.)
