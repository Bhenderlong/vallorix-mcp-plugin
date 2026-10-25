---
description: Fix a failing Vallorix compliance gap by generating the minimal code change and opening a pull request that references the control it satisfies
---

Fix the Vallorix gap identified by the argument (a measure ID, finding ID, or description).

## Steps

1. **Parse the target.** If the argument is a URL (e.g. `https://app.vallorix.ai/.../measures/<id>` or `.../findings/<id>`), extract the ID. Otherwise use it directly. If it's a description, call `listMeasures`/`listFindings` and fuzzy-match.
2. **Load remediation context** (follow the `remediate` skill's workflow): `getMeasure`/`getFinding` + `listMeasureControls` + `listMeasureEvidences` (+ `getControl`). Ground the fix in the control requirement, never general knowledge.
3. **Scan the repo** for the infrastructure/config that governs this control (Terraform, CloudFormation, CDK, Kubernetes/Helm, app config). Fix only the resources tied to the gap.
4. **Generate the minimal change.** Make only what's required to satisfy the control. Never weaken a security setting to make a check pass — if that's the only way, stop and tell the user.
5. **Propose, then open a PR.** Show the diff, then offer to create a branch and PR. In the PR body include attribution so the mapping is traceable and Vallorix can link the change back to the control:
   `Satisfies: Vallorix <measure|finding> <id> — <control code> (<framework>)`
   e.g. `Satisfies: Vallorix measure m_abc123 — CC6.1 (SOC 2)`.
6. **Close the loop.** After merge, remind the user to capture the evidence Vallorix expects (from `listMeasureEvidences`) and offer to update the item in Vallorix (`updateMeasure` state → `IMPLEMENTED`, or `updateFinding` status → `MITIGATED`/`CLOSED`) — with their confirmation.

## Edge cases
- **ID not found:** call `listMeasures`/`listFindings`, fuzzy-match, present the closest matches ("Did you mean…?"). Never dead-end.
- **Already implemented / closed:** "This measure is already `IMPLEMENTED`" (or the finding is `CLOSED`). Then show the open gaps so the user can pick another (`/vallorix:compliance-status`).
- **No IaC in this directory:** "I have the fix, but I don't see infrastructure code here." Offer: open Claude in the right repo, scaffold the config, or provide CLI steps.
- **IaC present but no matching resource:** "Found Terraform, but nothing here manages the resource this control governs." Offer: import + fix, fix elsewhere, or manual steps.
- **Policy/process gap (not code):** this isn't a code fix — hand off to `/vallorix:remediate` for the document/approval/access-review steps.
