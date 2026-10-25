---
description: Surface evidence that is missing, manual, or expiring across the organization's measures — so nothing lapses before an audit
---

Audit the evidence backing the organization's controls and flag what needs attention, from the user's own Vallorix instance.

## Steps

1. **Pull measures + their evidence.** Call `listMeasures` (focus on `IMPLEMENTED` ones — those are the controls currently claimed as met), then `listMeasureEvidences` for each to get the attached evidence with its **automation state** (`FULLY` / `PARTIALLY` / `AUTOMATION_READY` / `MANUAL`) and expiry/validity dates.
2. **Bucket the evidence:**
   - **Expiring / expired** — validity date passed or within 30 days. Highest priority: an implemented control with expired evidence will fail the audit.
   - **Manual, not yet automated** — `MANUAL` where an `AUTOMATION_READY` path exists: a candidate to connect a collector and stop doing it by hand.
   - **Missing** — an `IMPLEMENTED` measure with no evidence attached. That's a claim with no proof.
3. **Present a scannable report:**
   - Headline counts: "7 pieces of evidence expiring in 30 days · 12 still manual · 2 implemented controls with no evidence."
   - A table per bucket: measure, evidence, state, expiry, framework(s) it supports.
4. **Recommend actions:** refresh the expiring items first; for `AUTOMATION_READY` manual evidence, suggest connecting the relevant collector in Vallorix so it stays fresh automatically; for missing evidence, point to the measure that needs it.
5. **End with the next step:** "Want me to draft what to collect for the 2 unproven controls, or focus on the expiring batch first?"

## Edge cases
- **All evidence fresh and automated:** "All implemented controls have current evidence; nothing expiring in 30 days. Clean." 
- **Large volume:** group by framework and summarize; show the top at-risk items with "[N] more — filter by framework?"
- **No implemented measures yet:** point the user to `/vallorix:framework-readiness` to see what to implement first.
