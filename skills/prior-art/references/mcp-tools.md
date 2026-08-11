# NVENTOR patent MCP tools

The NVENTOR API exposes its patent search surface as an MCP server (`/mcp`, same API-key
auth). These four read-only tools are the primary way this skill searches. Depending on the
client, they may appear namespaced (e.g. `mcp__plugin_nventor_patents__search_patents` in
Claude Code, or under the connector name in claude.ai) — match on the trailing tool name.

If the tools are not available in the session, see `http-fallback.md` for the raw HTTP
equivalents (Claude Code / Codex only; claude.ai cannot make raw HTTP calls).

## search_patents

Runs a Lucene/Solr search. The server applies the right defaults — US-only filter,
relevance sort, standard field list — so you only supply the query itself.

| Input | Meaning |
|---|---|
| `query` (required) | Lucene query per `query-guide.md` — ALL essence concepts, OR-grouped synonyms |
| `rows` | results to return (default 100, max 500) |
| `start` | pagination offset |
| `published_before` | `YYYYMMDD` — prior-art cutoff for Mode B (filters `pd`) |
| `include_non_us` | default false; leave it that way unless the user asks |

Returns `{num_found, start, docs: [{ucid, title, abstract, cpc, publication_date}]}`.
Docs are relevance-ordered. If the backend deployment predates inline-field support, docs
carry only `ucid` and the result includes a `note` telling you to hydrate via
`fetch_patent_text` — follow it.

## lookup_patent

Resolves a publication number in any format ("US 9,162,553 B2", "US 2011/0042995 A1", a
UCID) to its UCID(s). Kind codes are handled server-side. Errors if nothing matches —
unpublished applications are not in the index, and that's expected (use pasted claims
instead).

## fetch_patent_text

Fetches full documents for up to **100 UCIDs in one call** — always batch, never loop.

| Input | Meaning |
|---|---|
| `ucids` (required) | up to 100 |
| `include_claims` | default true |
| `include_description` | default false — large; only for a close read of one patent |

Returns cleaned text (XML stripped): `{ucid, kind, document_type, publication_date,
abstract, claims?, description?}` per document. `document_type` is `grant` (B kinds) or
`application` (A kinds — may be pending, abandoned, or rejected); use it for the report's
grants-vs-applications split.

## validate_query

Checks a query's field names against the live field registry without executing it (catches
e.g. `cpci` — the real field is `cpc`). Free — use it whenever you're unsure a query is
well-formed, before spending a metered search.

## Budget

`search_patents` and `fetch_patent_text` hit the metered patent-data provider. The caps in
SKILL.md (3 searches, 2 text fetches per assessment) apply to these tools; `lookup_patent`
and `validate_query` are cheap and don't count.
