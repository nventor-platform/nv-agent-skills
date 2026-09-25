# NVENTOR patent MCP tools

The NVENTOR API exposes its patent search surface as an MCP server (`/mcp`, same API-key
auth). These five read-only tools are the primary way this skill searches. Depending on the
client, they may appear namespaced (e.g. `mcp__plugin_nventor_patents__search_patents` in
Claude Code, or under the connector name in claude.ai) — match on the trailing tool name.

If the tools are not available in the session, see `http-fallback.md` for the raw HTTP
equivalents (Claude Code / Codex only; claude.ai cannot make raw HTTP calls).

## search_patents

Runs a Lucene/Solr search. The server applies the right defaults — US-only filter,
relevance sort, standard field list — so you only supply the query itself.

| Input | Meaning |
|---|---|
| `query` (required) | Lucene query per `query-guide.md` — the product category plus one or two distinguishing elements, OR-grouped synonyms, scoped to `tac` by default (`ab_en` for precision, `text` for recall, `clm_en` for claims only) |
| `rows` | results to return (default 100, max 500). Use **50**: large pages exceed the client's inline tool-result limit and get written to a file you then have to grep |
| `start` | pagination offset |
| `published_before` | `YYYYMMDD` — prior-art cutoff for Patent mode (filters `pd`) |
| `include_non_us` | default false; leave it that way unless the user asks |
| `abstract_chars` | abstracts are truncated to 400 characters by default so pages stay inline; raise it only for a small page |

Returns `{num_found, start, docs: [{ucid, title, abstract, cpc, publication_date, filing_date, family_id}]}`.
Docs are relevance-ordered. If the backend deployment predates inline-field support, docs
carry only `ucid` and the result includes a `note` telling you to hydrate via
`fetch_patent_text` — follow it.

## lookup_patent

Resolves a publication number in any format ("US 9,162,553 B2", "US 2011/0042995 A1", a
UCID) to its UCID(s), returning per document `{ucid, title, document_type, filing_date,
publication_date, family_id, priority_applications}` — the filing date and priority
applications give you the prior-art cutoff without asking the user. Kind codes are handled server-side. Errors if nothing matches —
unpublished applications are not in the index, and that's expected (use pasted claims
instead).

## fetch_patent_text

Fetches full documents for up to **100 UCIDs in one call** — always batch, never loop. Claims
are long (often 3–8k characters each): fetch claims for your top ~12, not your whole shortlist,
or the result spills out of context.

| Input | Meaning |
|---|---|
| `ucids` (required) | up to 100 |
| `include_claims` | default true |
| `include_description` | default false — large; only for a close read of one patent |

Returns cleaned text (XML stripped): `{ucid, kind, document_type, publication_date,
abstract, claims?, description?}` per document. `document_type` is `grant` (B kinds) or
`application` (A kinds — may be pending, abandoned, or rejected); use it for the report's
grants-vs-applications split.

## get_citations

Follows the citation trail for up to 25 UCIDs.

| Input | Meaning |
|---|---|
| `ucids` (required) | your closest hits (and, in Patent mode, the source document) |
| `direction` | `backward` (default — what they cite), `forward` (later documents citing them), or `both` |
| `include_non_us` | default false |

Returns `{results: [{root, direction, citations: [{ucid, cited_by, title, publication_date,
family_id}]}]}`. `cited_by` is `examiner` (search-report citation), `applicant`, or another
source code. Titles let you triage without a text fetch; read the promising ones with
`fetch_patent_text`.

## validate_query

Checks a query's field names against the live field registry without executing it (catches
e.g. `cpci` — the real field is `cpc`). Free — use it whenever you're unsure a query is
well-formed, before spending a metered search.

## Budget

`search_patents` and `fetch_patent_text` hit the metered patent-data provider. The caps in
SKILL.md (3 searches, 2 text fetches, 2 citation lookups per assessment) apply to these tools; `lookup_patent`
and `validate_query` are cheap and don't count.
