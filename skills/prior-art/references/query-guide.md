# Patent query craft (Solr / IFI)

## Fields that matter

| Field | Meaning |
|---|---|
| `ucid` | Unique document ID, `US-5551212-A` |
| `ttl_en` | English title |
| `tac` | Title + abstract + claims (all languages) — **the default search field**: claim-level recall without description noise. Searchable only, never returned in results |
| `ab_en` | English abstract — high precision; use when `tac` is too broad. Returned in results, so also your reading substrate |
| `text` | Title + abstract + claims + description — maximum recall, low precision (omnibus filings dominate). Searchable only |
| `clm_en` | English claims only — searchable (e.g. `clm_en:(awning OR canopy)`), never returned in results; read claims via `fetch_patent_text` |
| `desc_en` | English description (rarely needed) |
| `cpc` | CPC classification codes (searchable; `cpci` is rejected by the field registry) |
| `ic` | IPC classification codes |
| `pd` | Publication date, YYYYMMDD |
| `pnctry` | Publication country (`fq: pnctry:US` always) |
| `pa` | Assignee/applicant name |

There are 178+ fields in the registry, but the set above covers this workflow. Friendly aliases like `abstract` and `title` are NOT real field names — use the codes above (check doubtful names with the `validate_query` tool).

## Operators (all supported)

```
OR-groups     tac:(awning OR canopy OR shade)
AND           tac:(vehicle OR car) AND tac:(door)
Wildcards     tac:deploy*            — deploy, deployed, deployment (no leading wildcards; no stemming, so the prefix must match the literal word)
NOT           tac:(vehicle NOT RV NOT camper)
Phrases       tac:"roof rack"
Proximity     tac:"door awning"~10   — words within 10 positions
Ranges        pd:[20020101 TO *]     — {} for exclusive bounds
CPC           cpc:B60J*
```

Advanced: `{!complexphrase inOrder=false}ab_en:"solar energy stor* modul*"~6` allows wildcards inside phrases.

## The query recipe

One broad query per assessment, shaped as:

```
<constraint-1 OR-group> AND <constraint-2 OR-group> AND <constraint-3 OR-group>
```

every term field-scoped to `tac`. Canonical example for "car door awning":

```
tac:(vehicle OR car OR automobile) AND tac:(door) AND tac:(awning OR canopy OR shade OR cover)
```

**Rules:**
- Every query MUST include ALL categorical constraints (essence concepts). Missing any = irrelevant results.
  - ✅ GOOD: `tac:(vehicle OR car) AND tac:door AND tac:(awning OR shade)`
  - ❌ BAD: `tac:(awning OR shade) AND tac:deploy*` — lost vehicle AND door
  - ❌ BAD: `tac:(door-actuated) AND tac:vehicle` — lost the shade/awning concept
- Feature terms (how it works) are optional broadeners — add them only if the constraint-only query is far too broad.
- Do NOT search for different products that share keywords (tonneau covers, rooftop tents, RV awnings are different products from a car-door awning). Exclude them with NOT when they pollute results.
- Synonyms must be technical/formal terms found in patent claims: near-synonyms and hypernyms. Never slang, never adjacent product categories, never terms so abstract they match everything ("transport", "device", "system" alone).

## Calibrating breadth

- `num_found < 20` → too narrow. Spend a second query: widen synonyms, add wildcards, swap a text constraint for a CPC code (`cpc:E04H15*`), or move from `tac` to `text` for description-level recall.
- Top rows off-category on `tac` → re-run the same query on `ab_en` for precision (abstract-level matching only).
- `num_found` in the hundreds → ideal. Top 100 relevance-sorted rows are your corpus.
- `num_found > ~2500` → very broad; results are still relevance-sorted and usable, but if the top rows are off-category, tighten with a NOT clause or an extra constraint.

Hard cap: **3 queries total** per assessment. The cap is enforced by discipline, not the server — hold yourself to it.
