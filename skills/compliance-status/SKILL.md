---
description: Show the user's open compliance gaps (unimplemented measures + open findings), prioritized by what can be fixed from this repository
---

Show the user where they are out of compliance, ranked by what this plugin can help fix. Everything here runs against the user's own Vallorix instance (the `VALLORIX_MCP_URL` — self-hosted or app.vallorix.ai); their evidence never leaves their boundary.

## Steps

1. **Fetch the gaps.** In parallel:
   - Call `listMeasures` and keep the ones whose `state` is **not** `IMPLEMENTED` and **not** `NOT_APPLICABLE` — i.e. `NOT_STARTED`, `IN_PROGRESS`, `NOT_IMPLEMENTED`, `UNKNOWN`. These are the control-implementation items still open.
   - Call `listFindings` and keep `status` `OPEN` or `IN_PROGRESS`. Note each finding's `priority` (HIGH/MEDIUM/LOW) and `kind` (MAJOR_NONCONFORMITY / MINOR_NONCONFORMITY / OBSERVATION / EXCEPTION).
   - Call `listFrameworks` so you can label each gap with the framework(s) it affects (SOC 2, ISO 27001, HIPAA, CMMC, …).
2. **Map gaps to frameworks.** For the top candidate measures, call `listMeasureControls` to see which controls (and therefore frameworks) each measure satisfies. A single unimplemented measure often blocks several frameworks at once — surface that.
3. **Categorize into fix-tiers** (mirror of how an engineer actually works):
   **Ready to fix from this repo** — the gap maps to infrastructure this repository manages. Detect deployment code: `provider "aws"`/`aws_*` in `.tf`, `provider "google"`/`google_*`, `provider "azurerm"`/`azurerm_*`, `AWSTemplateFormatVersion` (CloudFormation), `cdk.json` (CDK), `Dockerfile`/`k8s`/`helm` manifests, or app config/code matching the control (e.g. a measure about "enforce MFA" when an IdP/Terraform config is present). Use multiple signals. Present these first — one command: `/vallorix:fix <measureId|findingId>`.
   **Fixable with guidance** — code-remediable but the matching code isn't in this repo (different system/provider). Offer `/vallorix:remediate <id>` for a grounded plan.
   **Policy / process / manual** — needs a document, an approval, an access review, or a config change in another tool. The plugin gives guidance, not code.
4. **Present results — scannable, no raw API dumps.** For each tier, a table:
   - Gap (measure name or finding title)
   - ID
   - Type (Measure / Finding)
   - Priority / state (HIGH finding, or `NOT_STARTED` measure)
   - Framework(s) affected
   - For "Ready to fix": `Run /vallorix:fix <id>`
5. **Highlight cross-framework clusters.** If one measure unblocks multiple frameworks, call it out: "Implementing *Encryption at rest* closes gaps in SOC 2 CC6.1, ISO 27001 A.8.24, and HIPAA §164.312 at once — best bang for the buck."
6. **End with a clear next step:** "Want me to fix the top one?" or "Run `/vallorix:framework-readiness` to see how close you are to a specific framework."

## Edge cases
- **No open gaps:** "No open findings and all measures are implemented. Nice posture." Don't show an empty table.
- **Filter by framework** ("show SOC 2 gaps"): resolve the framework via `listFrameworks`/`getFramework`, then filter measures/findings to those mapped to it.
- **Filter by system** ("show AWS gaps"): filter to gaps whose controls/measures reference that integration; if none match, say so and show the full list.
- **"What should I fix first?":** rank by (1) IaC-fixable in this repo, (2) HIGH-priority / MAJOR_NONCONFORMITY, (3) cross-framework clusters, (4) how long it's been open. Lead with the cluster that closes the most framework requirements.
- **Very many gaps:** group by framework and summarize counts; show the top 5–10 highest-impact items with "[N] more — want the full list or a specific framework?"
