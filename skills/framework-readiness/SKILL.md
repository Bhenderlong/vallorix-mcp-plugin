---
description: Report how close the organization is to a given framework (SOC 2, ISO 27001, HIPAA, CMMC, …) — readiness %, implemented vs open controls, and the top blocking gaps
---

Answer "how ready are we for <framework>?" with a real number and the shortest path to green, computed from the user's own Vallorix instance.

## Steps

1. **Resolve the framework.** Call `listFrameworks`; match the user's term (SOC 2, ISO 27001, HIPAA, GDPR, PCI, NIST 800-171, CMMC, ISO 42001, DORA, …). If ambiguous, show the list and ask. If none named, show all frameworks with a one-line readiness number each.
2. **Pull the control set.** `getFramework` + `listControls` (scoped to that framework). For each control, use `listControlMeasures` to get the measures that implement it.
3. **Compute readiness.** A control counts as **met** when its measures are `IMPLEMENTED` (or `NOT_APPLICABLE` with a reason). Readiness % = met controls / total applicable controls. Also surface the measure-level view (implemented vs `NOT_STARTED`/`IN_PROGRESS`/`NOT_IMPLEMENTED`).
4. **Factor in findings.** Call `listFindings` and flag any `OPEN`/`IN_PROGRESS` finding mapped to this framework — an open MAJOR_NONCONFORMITY blocks certification even if the % looks high. List these as **blockers**.
5. **Present:**
   - Headline: "**SOC 2: 82% ready** — 41/50 controls met, 3 blocking findings."
   - A short table of the **top gaps** ordered by impact: control, the unimplemented measure, state, and whether it's fixable from this repo (`/vallorix:fix <id>`).
   - The **blockers** (open findings) separately.
6. **End with the shortest path:** "Closing these 4 measures would take you from 82% → 96%. Want me to start with the IaC-fixable ones?"

## Edge cases
- **No controls for the framework yet:** "That framework isn't set up in your Vallorix workspace. Add it in Vallorix, then re-run."
- **Everything implemented, no blockers:** "SOC 2: 100% of controls met, 0 open blockers. You're audit-ready on the Vallorix side — line up the auditor."
- **Cross-framework overlap:** when a measure spans multiple frameworks, note it so the user sees the leverage ("this one measure also moves ISO 27001 +4%").
