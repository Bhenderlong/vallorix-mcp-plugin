# Vallorix for Claude Code

Turn your compliance program into something you can drive from your editor. This plugin connects Claude Code to your **Vallorix** instance over MCP so you can ask *"where am I out of compliance?"*, get remediation grounded in your actual controls, and fix failing controls as code — without leaving your terminal.

**The difference from every hosted GRC copilot:** Vallorix is self-hostable, so the MCP endpoint points at **your** instance. Your controls, findings, and evidence never leave your infrastructure — the copilot runs against your sovereign deployment, not a vendor's cloud.

## What you can do

| Command | What it does |
|---|---|
| `/vallorix:compliance-status` | Lists your open gaps — unimplemented measures + open findings — ranked by what's fixable from this repo, grouped by framework. |
| `/vallorix:framework-readiness <framework>` | "How ready are we for SOC 2 / ISO 27001 / HIPAA / CMMC?" — readiness %, controls met vs open, and the shortest path to green. |
| `/vallorix:remediate <id>` | A remediation plan grounded in the real control requirement and the evidence Vallorix expects — security-safe, minimal change. |
| `/vallorix:fix <id>` | Scans your repo, generates the minimal fix, and opens a PR that references the control it satisfies. |
| `/vallorix:evidence-check` | Flags evidence that's expiring, still manual, or missing — so nothing lapses before an audit. |

You can also just talk to it: *"what's blocking our SOC 2?"*, *"fix the encryption-at-rest gap"*, *"what evidence expires this month?"*

## Setup

### 1. Point it at your Vallorix instance

The plugin reads the MCP URL from `VALLORIX_MCP_URL`, defaulting to the hosted app:

```bash
# Self-hosted / sovereign (recommended) — your own deployment:
export VALLORIX_MCP_URL="https://compliance.your-company.internal/api/mcp/v1"

# Or the hosted instance (default if unset):
# https://app.vallorix.ai/api/mcp/v1
```

The MCP server is a standard Streamable-HTTP MCP endpoint mounted at `/<your-vallorix-base>/api/mcp/v1`.

### 2. Authenticate with a Personal API Key

Create a key in Vallorix (**Settings → Personal API Keys**), scoped to what you want the copilot to touch, then set it:

```bash
export VALLORIX_API_KEY="<your-personal-api-key>"
```

The plugin sends it as a bearer token (`Authorization: Bearer $VALLORIX_API_KEY`). Scope and permissions follow the key's grants and your Vallorix role — the copilot can only see and change what the key allows. Revoke it anytime from the same screen.

> **Why a key and not browser OAuth?** The server fully supports OAuth 2.0 (PKCE, discovery, device flow), but its dynamic client registration is gated, so an API key is the zero-friction path that works on every instance. If your self-hosted instance opens registration, you can drop the `headers` block from `.mcp.json` and Claude Code will run the OAuth flow instead.

### 3. Install the plugin

Add this plugin to Claude Code (via your plugin marketplace or by cloning this directory into your plugins path), then run `/vallorix:compliance-status` to confirm the connection.

## How it works

The skills orchestrate Vallorix's MCP tools into an engineer-friendly loop:

- **Status** reads `listMeasures` (state) + `listFindings` (priority/kind) + `listFrameworks`, and maps gaps to frameworks via `listMeasureControls`.
- **Remediation** grounds every fix in `getMeasure`/`getFinding` + `listMeasureControls` + `listMeasureEvidences` (the real requirement and the evidence it needs) — never generic guesses, and never by weakening a security setting.
- **Fix** opens a PR attributed back to the control (`Satisfies: Vallorix measure <id> — CC6.1 (SOC 2)`), then offers to update the measure/finding state in Vallorix once it's done.

## Security & data residency

Nothing in this plugin sends your compliance data anywhere except your own Vallorix instance. Remediation plans are computed from your controls and evidence; code fixes stay in your repo and PRs. This is the whole point of running compliance on infrastructure you control.

---

*Vallorix, by Inferix · [vallorix.ai](https://vallorix.ai)*
