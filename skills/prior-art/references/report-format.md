# Report format & hedged-language rules

The deliverable is a **patent-landscape report**. Never call it an "opinion" — that implies legal advice this product does not provide.

**Mode B (existing patent / pending application):** title the report "Prior-Art Landscape for <publication number>", state the filing/priority cutoff used (or that none was available) directly under the title, and add one line noting the source document and suspected related filings were excluded. The verdict must never predict examination outcome, allowance, or validity. If no cutoff date was available, split "Most similar patents" into pre- and post-publication groups. Everything else below applies unchanged.

## Structure

```markdown
# Patent Landscape Report: <short idea name>

## Your invention, as assessed
<The confirmed idea statement, then: technical field, problem solved,
 the categorical constraints and features identified, suggested CPC codes.>

## Search summary
<Queries run (the actual Solr strings), results found per query,
 corpus size after dedup, how many patents were reviewed in depth.>

## Landscape assessment
- **Crowdedness**: sparse | moderate | crowded | dense | saturated
- **Confidence**: low | medium | high

### Key findings
<3–6 bullets. Each cites specific UCIDs and states factually what those
 patents cover. No value judgments about the user's idea.>

### Observed gaps
<What the search did NOT surface — framed strictly as observations about
 these search results, never as conclusions about the field.>

## Most similar patents
<Table, ranked by conceptual similarity:>
| # | Patent | Title / label | Date | Type | Signal | Why it matters |
<Type = Grant (B1/B2) or Application (A1/A2). Signal = strong/moderate/weak.
 "Why it matters" = the specific overlap or difference, one sentence.
 When the API provides no titles, use your abstract-derived label and note
 beneath the table that labels are descriptive, not official titles.>

## Verdict
<One paragraph. MUST start with "Based on this search…". Follows every
 language rule below.>
```

### Crowdedness scale

- **sparse** — few relevant patents found in search
- **moderate** — some relevant patents; appears to be room for differentiation
- **crowded** — many relevant patents in the space
- **dense** — heavy patent activity; limited white space remains
- **saturated** — core concept appears well-covered by existing patents

### Confidence

Base confidence on **patents actually reviewed**, not corpus size. A small corpus you read carefully justifies high confidence; a large corpus you skimmed does not.

## Hedged-language rules (mandatory, no exceptions)

You are not qualified to determine novelty or patentability, and this report must never read as if you did.

**Never say:**
- "is novel" / "is patentable" / "this combination is unique" / "no prior art exists"
- anything presenting a legal conclusion as fact

**Substitutions:**
| Instead of | Write |
|---|---|
| "is" | "appears to be" |
| "does not exist" | "the search did not find" |
| "proves" | "suggests" |
| "shows" | "may indicate" |
| unqualified claims | qualify with "based on this search" |

**Additional rules:**
- Key findings cite specific UCIDs and describe what the patents cover — factually, without judging the user's idea.
- Gaps are observations about *search results*: "Limited patents were found specifically combining X with Y" — NOT "the combination is novel."
- Distinguish grants (B1/B2) from applications (A1/A2 — may be pending, abandoned, or rejected). If results skew toward applications, say so in the verdict.
- The verdict must NOT recommend "further professional analysis" or "consulting a patent attorney" — this is implied, and stating it undermines the search.
- Scope every conclusion to this search: US publications, abstract-level matching, a bounded corpus.
