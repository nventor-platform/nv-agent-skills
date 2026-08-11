# NVENTOR Agent Skills

Skills that let your AI agent (Claude Code, claude.ai, Codex CLI, or any [Agent Skills](https://agentskills.io)-compatible tool) run NVENTOR patent workflows against the **NVENTOR patent MCP server** (`https://api.nventor.io/mcp`).

**Skills:**

- **`prior-art`** — guided prior-art search and patent-landscape assessment. Works on a new invention idea (guided clarify flow) or an existing patent / pending application (claim-derived, date-bounded search). Produces a hedged patent-landscape report using the MCP tools `search_patents`, `lookup_patent`, `fetch_patent_text`, and `validate_query`.

## Requirements

- An NVENTOR API key (issued per client; contact NVENTOR).
- A backend deployment that serves `/mcp` (nv-services-v2 ≥ the MCP release).

## Install — Claude Code

```
/plugin marketplace add nventor-platform/nv-agent-skills
/plugin install nventor@nventor
```

The plugin bundles the MCP server config. Set your key before launching:

```bash
export NVENTOR_API_KEY=nv-...
```

The skill triggers automatically on prior-art / patentability requests, or invoke it with `/nventor:prior-art`. The patent tools appear as `mcp__plugin_nventor_patents__*`.

## Use in claude.ai

1. Settings → **Connectors** → **Add custom connector** → URL `https://api.nventor.io/mcp`, and add your API key as an `X-API-Key` **request header**. (Header auth for connectors is in beta; if the option isn't visible, ask NVENTOR about access.)
2. Upload the skill: zip `skills/prior-art/` and add it under Settings → Capabilities (requires code execution enabled).

## Install — Codex CLI

Copy the skill into your Codex skills directory and register the MCP server per Codex's MCP configuration (same URL and `X-API-Key` header):

```bash
git clone https://github.com/nventor-platform/nv-agent-skills.git
cp -r nv-agent-skills/skills/prior-art ~/.codex/skills/
```

Without MCP configured, the skill falls back to the raw HTTP API via curl (`references/http-fallback.md`) using `NVENTOR_API_KEY`.

## Notes

- The skill never makes legal determinations — reports use hedged language and are not legal advice or a prediction of examination outcome.
- Searches consume metered patent-data credits; the skill caps itself at 3 searches per assessment.
