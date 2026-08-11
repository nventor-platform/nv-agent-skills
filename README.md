# NVENTOR Agent Skills

Skills that let your AI coding agent (Claude Code, Codex CLI, or any [Agent Skills](https://agentskills.io)-compatible tool) run NVENTOR patent workflows directly against the NVENTOR API.

**Skills:**

- **`prior-art`** — guided prior-art search and patent-landscape assessment. Works on a new invention idea (guided clarify flow) or an existing patent / pending application (claim-derived, date-bounded search). Produces a hedged patent-landscape report.

## Requirements

- An NVENTOR API key. Set it as the `NVENTOR_API_KEY` environment variable, or the skill will ask for it.
- Optional: `NVENTOR_API_URL` to override the API base URL (defaults to `https://api.nventor.io`).

## Install — Claude Code

```
/plugin marketplace add nventor-platform/nv-agent-skills
/plugin install nventor@nventor
```

The skill triggers automatically on prior-art / patentability requests, or invoke it directly with `/nventor:prior-art`.

## Install — Codex CLI

Codex reads the same skill format. Copy (or symlink) the skill into your Codex skills directory:

```bash
git clone https://github.com/nventor-platform/nv-agent-skills.git
cp -r nv-agent-skills/skills/prior-art ~/.codex/skills/
```

(Use `.codex/skills/` inside a project instead for project-scoped installs.) Invoke via `/skills` / `$prior-art`, or just describe a prior-art task.

## Notes

- The skill never makes legal determinations — reports use hedged language and are not legal advice or a prediction of examination outcome.
- Searches consume metered patent-data credits; the skill caps itself at 3 searches per assessment.
