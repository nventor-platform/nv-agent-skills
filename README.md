# NVENTOR Agent Skills

Run NVENTOR patent workflows — prior-art search and patent-landscape assessment — from your own AI agent (Claude Cowork, Claude Code, Codex CLI, or any [Agent Skills](https://agentskills.io)-compatible tool), powered by the **NVENTOR patent MCP server** (`https://api.nventor.io/mcp`). ChatGPT support is coming soon.

## Install: tell your agent to do it

Have your NVENTOR API key ready (`nv-…`, issued by NVENTOR), then paste this into your agent:

> Install the NVENTOR prior-art skill and patent connector from
> https://github.com/nventor-platform/nv-agent-skills — clone the repo and follow its INSTALL.md.

Your agent handles the rest — copying the skill into place, configuring the connector, and verifying — asking you only for the API key and permission to edit its config. [`INSTALL.md`](./INSTALL.md) contains the agent-facing instructions for every supported environment (Cowork/Desktop, Claude Code, Codex).

**Claude Code shortcut** (no agent-driven install needed):

```
/plugin marketplace add nventor-platform/nv-agent-skills
/plugin install nventor@nventor
export NVENTOR_API_KEY=nv-...   # then restart Claude Code
```

## What you get

- **`prior-art` skill** — two modes, one report:
  - **Idea mode**: describe an invention in plain language, answer a couple of clarifying questions, and get a hedged patent-landscape report — similar patents, crowdedness, observed gaps.
  - **Patent mode**: give a patent/application number or paste your own draft claims; the skill splits claim 1 into elements, applies the prior-art cutoff date, and adds an element-by-element coverage table.
  Both run faceted searches of the US patent corpus and follow the examiner-citation trail of the closest hits.
- **Patent MCP tools** — `search_patents`, `lookup_patent`, `fetch_patent_text`, `get_citations`, `validate_query` — available to your agent directly for ad-hoc patent questions outside the guided flow.

## Coming soon

- **ChatGPT** — its connector system speaks the same MCP protocol but requires OAuth, which the NVENTOR API doesn't expose yet. The same skill and tools carry over unchanged once it lands.
- **claude.ai (web)** — gated on the same OAuth work (or Anthropic's header-auth beta reaching your account).

## Notes

- Reports use hedged language and are not legal advice, a legal opinion, or a prediction of examination outcome.
- Searches consume metered patent-data credits; the skill caps itself at 3 searches, 2 text fetches, and 2 citation lookups per assessment.
- Keys are issued per client and revocable by NVENTOR; keep yours out of chats and repos.
