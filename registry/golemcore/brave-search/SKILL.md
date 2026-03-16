---
name: brave-search
description: Use the brave_search tool effectively for fast web discovery, targeted lookup, and query-operator-driven search refinement.
available: true
model_tier: smart
---

Use this skill when the task needs fresh web discovery, a shortlist of relevant sources, or quick targeted lookup through the `brave_search` tool.

Do not use Brave Search for:
- repo-local facts already present in the workspace
- broad synthesis when a grounded answer with richer filters would be better
- tasks that need page extraction rather than result discovery

## Tool Summary

`brave_search` takes:
- `query` required
- `count` optional from 1 to 20

The tool returns:
- a human-readable result list with titles, URLs, and short descriptions
- structured data with `query`, `count`, and `results[]`
- per-result fields such as `title`, `url`, and `description`

Brave Search is intentionally simple. It does not expose domain filters, recency filters, or answer synthesis parameters, so query quality matters much more.

## Search Workflow

1. Define the lookup job before searching.
   Decide whether you need:
   - one authoritative source
   - a quick list of candidate sources
   - confirmation of a specific claim
   - current coverage on a recent event

2. Write a precise, operator-friendly query.
   Prefer compact, information-dense queries over conversational prompts.
   Good:
   - `site:ai.google.dev thought signatures gemini api`
   - `openai responses api background mode docs`
   - `site:sec.gov 10-k microsoft 2025`
   Bad:
   - `please search the web and tell me about`

3. Keep result count tight.
   - Default to `count=3` to `5`.
   - Use `6` to `8` when comparing vendors or conflicting claims.
   - Avoid large result sets unless you are doing discovery and plan to refine immediately.

4. Refine with query operators instead of over-searching.
   Since the tool only exposes `query` and `count`, use search syntax directly:
   - `site:example.com` to constrain domains
   - quoted phrases for exact wording
   - product versions, dates, and spec names for precision
   - `filetype:pdf` when you want filings, reports, or standards PDFs
   - `intitle:` when the title signal matters

5. Follow up with narrower searches.
   If the first result set is noisy:
   - add the official domain
   - add the exact product or version string
   - add the month or year
   - split one broad question into two smaller searches

## Query Patterns

### Official docs lookup

Example:

```json
{
  "query": "site:docs.python.org asyncio timeout",
  "count": 3
}
```

### Claim verification

Search the exact claim, then the claimant, then the date/version.

Example:

```json
{
  "query": "\"thought_signature\" gemini tool calling March 2026",
  "count": 5
}
```

### Current news discovery

Because the tool has no explicit news mode, bake freshness into the query.

Example:

```json
{
  "query": "gemini 3.1 flash lite preview March 2026 announcement",
  "count": 5
}
```

### Source shortlist building

Use a broader first pass, then re-run focused queries on the most promising sources.

## Source Handling Rules

- Cite the returned URLs or source titles, not "Brave Search".
- Prefer official docs, primary announcements, standards bodies, regulators, and maintainers over reposts.
- Do not infer facts from ranking alone; inspect the actual result text and domain.
- If the search result descriptions are too thin, run a follow-up search with a more constrained query instead of bluffing.

## Practical Heuristics

- Make the query smarter before making it longer.
- Use `site:` aggressively for official documentation.
- Add exact dates and versions when the topic is time-sensitive.
- If you need a grounded answer with rich citations, Brave Search is often the first pass, not the last pass.
- Two focused searches usually outperform one vague search.

## Output Contract

After using `brave_search`:
- answer the user's question directly
- mention the most relevant sources
- call out uncertainty when the result set is weak or conflicting
- say what follow-up query would improve confidence if needed
