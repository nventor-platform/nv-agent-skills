# INSTALL.md — instructions for the AI agent performing this install

You are an AI agent (Claude Cowork, Claude Code, Codex, or similar) that has been asked to
install the NVENTOR prior-art skill and patent connector from this repository. Follow the
section for the environment you are running in. Ask the user only for the things listed
under "What you need from the user" — everything else is yours to do.

## What you need from the user

1. **Their NVENTOR API key** (starts with `nv-`). Ask for it when you reach the connector
   step. NEVER echo the key back, write it into chat, or store it anywhere other than the
   config/env locations these instructions specify.
2. **Approval to edit their MCP/desktop config file** before you modify it.

## What gets installed

- The **`prior-art` skill** (`skills/prior-art/` in this repo) — a guided prior-art patent
  assessment workflow.
- The **NVENTOR patent MCP connector** — `https://api.nventor.io/mcp`, authenticated with
  the user's API key via an `X-API-Key` header. It provides the tools `search_patents`,
  `lookup_patent`, `fetch_patent_text`, `get_citations`, and `validate_query` that the skill drives.

## Getting the repo contents

Clone `https://github.com/nventor-platform/nv-agent-skills.git`. The repo is public, so no
credentials are needed. If your environment can't run `git`, have the user download the
repo as a zip from GitHub in their browser and give you the file — then unzip and continue.

## Claude Code

Don't install manually — the plugin system does everything:

1. Tell the user to run:
   ```
   /plugin marketplace add nventor-platform/nv-agent-skills
   /plugin install nventor@nventor
   ```
2. Have them set `NVENTOR_API_KEY` in their environment (shell profile) and restart
   Claude Code. The bundled `.mcp.json` auto-configures the connector.

## Claude Cowork / Claude Desktop

Two pieces: the skill (a folder copy) and the connector (a config edit).

**Skill:**
1. Locate the user's connected Skills folder (ask if you can't determine it).
2. Copy this repo's `skills/prior-art/` directory into it, keeping the folder name
   `prior-art`. New sessions pick it up automatically.

**Connector** (requires the user's API key and their approval):
1. Locate the Claude Desktop config file:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
2. Back it up, then **merge** (never overwrite existing entries) this server into its
   `mcpServers` object, substituting the user's real key:
   ```json
   {
     "mcpServers": {
       "nventor-patents": {
         "command": "npx",
         "args": ["-y", "mcp-remote", "https://api.nventor.io/mcp",
                  "--header", "X-API-Key:${NVENTOR_API_KEY}"],
         "env": { "NVENTOR_API_KEY": "<the user's nv- key>" }
       }
     }
   }
   ```
   (`npx`/Node is required for the `mcp-remote` bridge — check it's available and tell the
   user if not.)
3. Tell the user to fully restart the Claude desktop app.
4. If you cannot reach or edit the config file from your sandbox, show the user the exact
   JSON above (with a placeholder, not their real key if it was given in chat) and walk
   them through pasting it themselves.

## Codex CLI

1. Copy `skills/prior-art/` into `~/.codex/skills/` (or `.codex/skills/` in the project).
2. Register the MCP server per the user's Codex version (same URL and `X-API-Key` header),
   or skip it — the skill has a raw-HTTP fallback (`references/http-fallback.md`) that
   only needs `NVENTOR_API_KEY` exported in the environment.

## Verify

In a fresh session, confirm the install by (a) checking the patent tools are
available, and (b) asking a prior-art question (e.g. "run a quick prior-art check on a
collapsible car-door awning") — the skill should trigger and call `search_patents`. An
auth error means the key is wrong or revoked: have the user re-check it with NVENTOR;
don't retry.

## Uninstall

Remove the `prior-art` folder from the skills location and the `nventor-patents` entry
from the MCP config.
