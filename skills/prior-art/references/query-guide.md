# Patent query craft (Solr / IFI)

## Fields that matter

| Field | Meaning |
|---|---|
| `ucid` | Unique document ID, `US-5551212-A` |
| `ttl_en` | English title |
| `ab_en` | English abstract — your primary search & reading substrate |
| `clms_en` | English claims (fetch via `/api/ifi/text`, not in bulk search `fl`) |
| `desc_en` | English description (rarely needed) |
| `cpc` | CPC classification codes (searchable; `cpci` is rejected by the field registry) |
| `ic` | IPC classification codes |
| `pd` | Publication date, YYYYMMDD |
| `pnctry` | Publication country (`fq: pnctry:US` always) |
| `pa` | Assignee/applicant name |

There are 178+ fields (`GET /api/ifi/fields`), but the set above covers this workflow. Friendly aliases like `abstract`, `title`, `cpc` are NOT real field names — use the codes above.

## Operators (all supported)

```
OR-groups     ab_en:(awning OR canopy OR shade)
AND           ab_en:(vehicle OR car) AND ab_en:(door)
Wildcards     ab_en:deploy*          — deploy, deployed, deployment (no leading wildcards)
NOT           ab_en:(vehicle NOT RV NOT camper)
Phrases       ab_en:"roof rack"
Proximity     ab_en:"door awning"~10 — words within 10 positions
Ranges        pd:[20020101 TO *]     — {} for exclusive bounds
CPC           cpc:B60J*
```

Advanced: `{!complexphrase inOrder=false}ab_en:"solar energy stor* modul*"~6` allows wildcards inside phrases.

## The query recipe

One broad query per assessment, shaped as:

```
<constraint-1 OR-group> AND <constraint-2 OR-group> AND <constraint-3 OR-group>
```

every term field-scoped to `ab_en`. Canonical example for "car door awning":

```
ab_en:(vehicle OR car OR automobile) AND ab_en:(door) AND ab_en:(awning OR canopy OR shade OR cover)
```

**Rules:**
- Every query MUST include ALL categorical constraints (essence concepts). Missing any = irrelevant results.
  - ✅ GOOD: `ab_en:(vehicle OR car) AND ab_en:door AND ab_en:(awning OR shade)`
  - ❌ BAD: `ab_en:(awning OR shade) AND ab_en:deploy*` — lost vehicle AND door
  - ❌ BAD: `ab_en:(door-actuated) AND ab_en:vehicle` — lost the shade/awning concept
- Feature terms (how it works) are optional broadeners — add them only if the constraint-only query is far too broad.
- Do NOT search for different products that share keywords (tonneau covers, rooftop tents, RV awnings are different products from a car-door awning). Exclude them with NOT when they pollute results.
- Synonyms must be technical/formal terms found in patent claims: near-synonyms and hypernyms. Never slang, never adjacent product categories, never terms so abstract they match everything ("transport", "device", "system" alone).

## Calibrating breadth

- `numFound < 20` → too narrow. Spend a second query: widen synonyms, add wildcards, or swap a text constraint for a CPC code (`cpc:E04H15*`).
- `numFound` in the hundreds → ideal. Top 100 relevance-sorted rows are your corpus.
- `numFound > ~2500` → very broad; still usable with `sort: score desc`, but if the top rows are off-category, tighten with a NOT clause or an extra constraint.

Hard cap: **3 queries total** per assessment. The cap is enforced by discipline, not the server — hold yourself to it.
