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

## The faceted recipe

Three searches per assessment, each shaped as `<category OR-group> AND <element OR-group> [AND <element OR-group>]`,
every term field-scoped to `tac`. Worked example — "an awning mounted above a car door that unrolls
automatically when the door opens, driven by a spring-loaded roller":

```
Core     tac:(vehicle OR car OR automobile) AND tac:(awning OR canopy OR shade) AND tac:(door)
Facet A  tac:(vehicle OR car OR automobile) AND tac:(awning OR canopy OR shade) AND tac:(automatic* OR motor* OR actuat* OR sensor*)
Facet B  tac:(vehicle OR car OR automobile) AND tac:(awning OR canopy OR shade) AND tac:(roller OR spring* OR retract*)
```

Each facet keeps the **category** (vehicle awning) and swaps in a different **distinguishing element**. A
reference with a spring roller awning that isn't tied to the door — exactly the kind an examiner combines with
another reference — is found by Facet B even though a single everything-AND query would have excluded it.

**Rules:**
- Every query includes the product category. Category-free queries return off-category noise.
  - ✅ `tac:(vehicle OR car) AND tac:(awning OR shade) AND tac:(roller OR spring*)`
  - ❌ `tac:(awning OR shade) AND tac:(roller OR spring*)` — lost the vehicle; matches patio and window awnings
- Keep each query to 2–3 AND-groups. Four or more AND-groups usually over-constrain to a handful of near-duplicates.
- Synonyms must be the technical/formal terms used in patent claims: near-synonyms and hypernyms. Never slang,
  never adjacent product categories, never terms so abstract they match everything ("device", "system" alone).
- Exclude different products that share keywords with `NOT` when they pollute results (a car-door awning search
  should exclude tonneau covers and rooftop tents).
- Wildcards are your stemming: the index has none, so `retract*` is needed to catch retractable/retracting/retraction.

## Calibrating breadth

- `num_found < 10` → too narrow. Widen synonyms, add wildcards, drop to two AND-groups, or move from `tac` to `text`.
- Top rows off-category → add `NOT` exclusions, or re-scope the category group to `ab_en` for precision.
- `num_found` in the tens to low thousands → fine; results are relevance-sorted, so the top 50 are your slice.
- If the three facets overlap heavily (same top results), the next search should be more different, not more of the same:
  a CPC-scoped version (`cpc:A63B*`) or a `text`-field version of the core.

Hard cap: **3 searches** per assessment. The cap is enforced by discipline, not the server — plan the three before
running the first.
