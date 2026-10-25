---
description: Produce a grounded remediation plan for a Vallorix compliance gap (measure or finding), using the real control requirement and evidence context
---

# Vallorix Remediation

Help the user close a compliance gap — an unimplemented measure or an open finding — with a plan grounded in what Vallorix actually requires, not generic LLM guesses. All context is pulled from the user's own Vallorix instance.

## Key tools (from the Vallorix MCP)

- `getMeasure` / `getFinding` — the gap itself: description, state/status, priority.
- `listMeasureControls` — which controls (and frameworks) the measure satisfies. **This is the "why" — it tells you the exact requirement the fix must meet.**
- `listMeasureEvidences` — what evidence currently proves (or fails to prove) the measure, incl. its automation state (FULLY / PARTIALLY / AUTOMATION_READY / MANUAL) and expiry. The fix usually needs to produce or refresh one of these.
- `listMeasureTasks` — any tasks already defined for the measure.
- `getControl` — the control's framework language, so remediation maps to the actual clause (e.g. SOC 2 CC6.1, ISO 27001 A.8.24).
- `listFindingAudits` — history/context for a finding.

## Response principles (apply to every remediation)

1. **Always load the real context first.** Call `getMeasure`/`getFinding` **and** `listMeasureControls` before proposing anything. Ground the fix in the control requirement, never in general knowledge alone.
2. **Never weaken security.** Do not disable encryption, remove access controls, open a group to `0.0.0.0/0`, drop logging/MFA, or trade security for convenience. If a fix seems to require that, stop and flag it.
3. **Minimal change only.** Make exactly what's needed to satisfy the control. No refactors, no cleanup.
4. **Supplement stale guidance with a web search.** If the control or finding references an external console/tool/provider, web-search current docs — platform guidance ages fast.
5. **Say what you can't do.** If there's no matching code, say so plainly and give the manual steps + the config to change.
6. **Close the loop in Vallorix.** After the fix, name the evidence the user should attach (`listMeasureEvidences` tells you what's expected) and offer to update the item: `updateMeasure` (state → `IMPLEMENTED`) or `updateFinding` (status → `MITIGATED`/`CLOSED`) — only with the user's confirmation.
7. **Scannable output.** Tables for lists, code blocks for diffs/commands, bold for the control it satisfies.
8. **Sovereign by default.** Remind the user when relevant: this ran entirely against their Vallorix instance — no evidence or control data left their infrastructure.

## Core workflow

1. **Resolve the target.** Accept a measure ID, finding ID, or a description. If it's a description, call `listMeasures`/`listFindings` and fuzzy-match; if several match, show candidates and ask.
2. **Load context:** `getMeasure`/`getFinding` + `listMeasureControls` + `listMeasureEvidences` (+ `getControl` for the clause language).
3. **State the requirement** in one line: "Control **CC6.1** requires encryption at rest on all data stores; this measure is `NOT_IMPLEMENTED`."
4. **Scan the repo** for IaC/config matching the requirement (Terraform/CloudFormation/CDK/K8s/app config).
5. **Produce the plan:** the minimal change (diff if code is present), the evidence to capture, and the control/framework it closes.
6. **Offer to apply:** `/vallorix:fix <id>` to generate the PR, or walk the manual steps. Then offer to update the measure/finding state in Vallorix.
