# NVENTOR Agent Skills

Skills that let your AI agent (Claude Code, Claude Cowork, claude.ai, Codex CLI, or any [Agent Skills](https://agentskills.io)-compatible tool) run NVENTOR patent workflows against the **NVENTOR patent MCP server** (`https://api.nventor.io/mcp`). ChatGPT support is coming soon.

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

## Install — Claude Cowork (desktop)

Cowork picks up skills from the Skills folder you've connected to it — no upload or restart needed:

1. Clone this repo (or download it) and copy `skills/prior-art/` into your connected Skills folder. Start a new session and ask a prior-art question — the skill triggers automatically.
2. Give Cowork the patent tools by adding the MCP server to your Claude Desktop config (`claude_desktop_config.json`), with your API key in the header via the `mcp-remote` bridge:

```json
{
  "mcpServers": {
    "nventor-patents": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://api.nventor.io/mcp",
               "--header", "X-API-Key:${NVENTOR_API_KEY}"],
      "env": { "NVENTOR_API_KEY": "nv-your-key-here" }
    }
  }
}
```

3. Restart the desktop app once after editing the config; the tools then appear in every Cowork session.

This route needs no OAuth and no beta features — it works on any current Claude Desktop/Cowork install.

## ChatGPT — coming soon

ChatGPT support (via its connector system, which speaks the same MCP protocol) is planned. The blocker is OAuth: ChatGPT connectors require an OAuth flow rather than API-key headers, which the NVENTOR API doesn't expose yet. Watch this repo — the same skill and tools will carry over unchanged once it lands.

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
