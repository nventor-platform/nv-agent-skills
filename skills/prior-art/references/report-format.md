# Report format & hedged-language rules

The deliverable is a **patent-landscape report**. Never call it an "opinion" — that implies legal advice this product does not provide.

There is one template with two mode-specific additions. Idea mode uses it as is; Patent mode adds the
element-coverage table and date framing.

## Template

```markdown
# Patent Landscape Report: <short invention name>
<Patent mode: "Prior-art landscape for <publication number or 'your draft'>", then on the next line:
 "Cutoff: prior art published before <YYYY-MM-DD> (<source of the date>)" — or state that no cutoff was
 available. Add one line noting the source document and its family were excluded.>
<If you skipped confirmation because the user asked you to just run it: one "Assumptions" line listing the
 statement/cutoff you assumed.>

## The invention, as assessed
<Idea mode: the confirmed idea statement. Patent mode: a 2–3 sentence summary plus the claim-1 element list
 (E0 preamble, E1…En). Then: technical field, problem solved, product category, distinguishing elements,
 CPC subclasses considered.>

## Search summary
<The three queries actually run (the Solr strings) with results found for each, candidates after dedup,
 and how many were read in depth (abstract) and in detail (claims).>

## Landscape assessment
- **Crowdedness**: sparse | moderate | crowded | dense | saturated
- **Confidence**: low | medium | high

### Key findings
<3–6 bullets. Each cites specific UCIDs and states factually what those documents cover. No value judgments
 about the user's invention.>

### Element coverage (Patent mode only)
| Element | Appears in | Notes |
|---|---|---|
| E1 <short name> | US-…, US-… | <one line: how closely, or "not observed in the documents read"> |
<One row per claim-1 element. List only references you actually read. This shows which elements the search
 found and which it did not. It is not a validity or obviousness analysis, and should not be worded as one.>

### Observed gaps
<What the search did NOT surface — framed strictly as observations about these results, never conclusions
 about the field.>

## Most similar documents
| # | Patent | Title / label | Date | Type | Signal | Why it matters |
|---|---|---|---|---|---|---|
<Ranked by conceptual similarity, 10–15 rows. Type = Grant (B1/B2) or Application (A1/A2). Signal =
 strong/moderate/weak. "Why it matters" = the specific overlap or difference, one sentence (Patent mode: name
 the elements it appears to disclose, e.g. "E1, E3"). Use the returned title; if none, an abstract-derived
 label, noted beneath the table.>

## Verdict
<One paragraph. MUST start with "Based on this search…". Follows every language rule below.>
```

Patent mode with no cutoff available: split "Most similar documents" into two tables, published before and
after the source document's publication date.

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
- Element coverage says a reference "appears to describe" an element — never that an element or claim "is anticipated", "is obvious", or "is invalid".
- Scope every conclusion to this search: US publications, abstract- and claim-level reading, a bounded corpus.
