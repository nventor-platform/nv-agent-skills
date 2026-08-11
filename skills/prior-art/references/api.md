# NVENTOR API reference (client surface)

Base URL: `NVENTOR_API_URL` env var, else `https://api.nventor.io`. Local dev: `http://localhost:8080`.

**Auth**: every `/api/*` route requires the API key in the `X-API-Key` header. `/health` and `/swagger/*` are open. A 401/403 means the key is invalid — stop; it will fail identically on every endpoint.

Interactive docs: `{base}/swagger/`.

## Endpoints

| Method & path | Purpose |
|---|---|
| `GET /health` | Server health (no auth) |
| `GET /api/ifi/status` | Patent-source (IFI) connectivity + latency |
| `POST /api/ifi/search` | Solr patent search — the workhorse |
| `POST /api/ifi/search/validate` | Validate a query without spending a search |
| `GET /api/ifi/search/schema` | Solr field schema |
| `GET /api/ifi/fields` | List all searchable fields with properties |
| `POST /api/ifi/text` | Fetch full documents (claims, description) for up to 100 UCIDs |
| `GET /api/ifi/text/{ucid}` | Same, single UCID |

## POST /api/ifi/search

Body uses **raw Solr parameter names**:

```json
{
  "q": "ab_en:(vehicle OR car OR automobile) AND ab_en:(door) AND ab_en:(awning OR canopy OR shade OR cover)",
  "fl": ["ucid", "ttl_en", "ab_en", "cpc", "pd"],
  "rows": 100,
  "start": 0,
  "fq": ["pnctry:US"],
  "sort": ["score desc"]
}
```

Non-negotiable conventions (the server does NOT default these for you):

- **`rows` defaults to 10** if omitted (clamped to max 1000). Always set it explicitly — 100 is the standard corpus fetch.
- **`fq: ["pnctry:US"]`** — always filter to US publications; that's the product's scope.
- **`sort: ["score desc"]`** — without it results come back newest-first, not most-relevant-first.
- **`fl`** — request `ucid, ttl_en, ab_en, cpc, pd` (note: `cpc`, NOT `cpci` — the registry rejects `cpci` with a 400). Do NOT request `clms_en` in bulk searches.
- Field names in `q`/`fl` are validated against a registry; an unknown field returns 400 with details.
- `q` is required; server-side timeout is 30s.

**What the docs actually contain — check, don't assume.** As of 2026-08, `/api/ifi/*` routes to the hosted IFI index on **both** dev and prod, and IFI's search index doesn't store content fields: docs contain **`ucid` only**, regardless of `fl` (verified live). You must hydrate via `POST /api/ifi/text` (see below), and **titles are not available at all** — identify patents by UCID and a short label you derive from the abstract.

Still, write the adaptive check: if the first doc comes back with `ab_en`/`ttl_en` inline, the API has been rewired to NVENTOR's Solr warehouse — use those fields directly and skip hydration for abstracts (you'll still want `/api/ifi/text` for claims on your shortlist).

### Response

```json
{
  "status": "success",
  "time": "0.048",
  "content": {
    "responseHeader": { "status": 0, "QTime": 44, "params": { } },
    "response": {
      "numFound": 1763,
      "numFoundExact": true,
      "start": 0,
      "maxScore": 17.4,
      "docs": [
        { "ucid": "US-9162553-B2" }
      ]
    }
  }
}
```

Parsing notes:
- Docs are already in relevance order when you passed `sort: ["score desc"]`.
- In warehouse mode, Solr multi-valued fields may arrive as **arrays of strings** (`ttl_en`, `ab_en` often do). If a value is an array, join/take the first element.
- `pd` is a YYYYMMDD integer/string.
- UCID format is `COUNTRY-NUMBER-KIND`, e.g. `US-9162553-B2`. The kind code tells you grant vs application: **B1/B2 = granted patent; A1/A2 = published application** (may be pending, abandoned, or rejected).
- Dedupe by UCID across queries; after hydration, also dedupe identical abstracts (patent families republish the same abstract) and drop docs with no abstract.

### Looking up a specific patent by number

To resolve a publication number to a UCID (Mode B intake), search on `pnnum` with the digits only — no kind code needed, and it works in every backend mode:

```json
{ "q": "pnnum:9162553", "fl": ["ucid"], "rows": 5 }
```

Verified: returns the matching UCID(s) (e.g. `US-9162553-B2`). If multiple kinds come back (A1 application + B2 grant of the same filing), prefer the one the user named, else the grant. Then fetch content via `GET /api/ifi/text/{ucid}`. Note: `fam` (family ID), `ad` (filing date), and `pd` are valid registry fields but are NOT returned in search docs in hosted-IFI mode — filing dates must come from the user, and family exclusion is done by abstract matching.

For date-bounded prior-art searches (Mode B), add the cutoff as a filter query: `"fq": ["pnctry:US", "pd:[* TO 20240315]"]`.

### Zero results

Recovery tactics, in order: (1) add wildcards — `deploy*` instead of `deploy`; (2) widen OR-groups with more synonyms; (3) proximity — `ab_en:"vehicle shade"~15`; (4) drop a low-priority feature term (never a categorical constraint).

## POST /api/ifi/text

```json
{ "ucids": ["US-9162553-B2", "US-6044856-A"] }
```

1–100 UCIDs per request; 60s timeout. One call for your whole hydration set — never per-patent `GET /api/ifi/text/{ucid}` calls in a loop.

**The response is large (~20 KB per document — it always includes the full description). NEVER read the raw response into the conversation.** Save it to a file and extract just what you need with a script. Verified response shape (Go-serialized, capitalized keys):

```json
{
  "Status": "...", "Time": "...",
  "Documents": [
    {
      "UCID": "",                 // empty — reconstruct from DocumentID below
      "Title": "",                // empty — titles are NOT available from this endpoint
      "BibliographicData": {
        "PublicationRef": { "DocumentID": {
          "Country": "US", "DocNumber": "9162553", "Kind": "B2", "Date": "20151020" } }
      },
      "AbstractData":    { "Content": "<p ...>abstract text with XML/HTML tags</p>" },
      "ClaimsData":      { "Content": "<claim-statement>...</claim-statement><claim ...>" },
      "DescriptionData": { "Content": "...very large, usually skip..." }
    }
  ]
}
```

Extraction script pattern (adapt as needed):

```python
import json, re
strip = lambda s: re.sub(r"<[^>]+>", " ", s or "").strip()
docs = json.load(open("text.json"))["Documents"]
for d in docs:
    did = d["BibliographicData"]["PublicationRef"]["DocumentID"]
    ucid = f'{did["Country"]}-{did["DocNumber"]}-{did["Kind"]}'
    print(ucid, did["Date"], did["Kind"], "|", strip(d["AbstractData"]["Content"])[:600])
```

Print claims (`ClaimsData.Content`, tags stripped) only for your shortlist; ignore `DescriptionData` unless a specific patent demands a closer look.

## POST /api/ifi/search/validate

Same `q`/`fl`/`fq` shape as search; checks syntax and field names without executing. Use it if you're unsure a complex query parses — it doesn't count against your search budget.

## Errors

Error envelope: `{ "status": "error", "timestamp": "...", "error": "...", "message": "...", "details": "..." }`. Validation failures return `{ "invalid": { "field": ["problem"] } }`.

- **400** — malformed body or unknown field name; fix and retry (doesn't count as a spent search if it never executed... but prefer `/validate` for anything doubtful).
- **401/403** — bad API key. Stop and tell the user.
- **503** — patent source not configured/available; report and stop.
- Timeouts/5xx — retry once, then surface the error.
