---
name: tavily-search
description: Use the tavily_search tool effectively for fresh web information, source-backed answers, domain-constrained lookup, and iterative search refinement.
available: true
model_tier: smart
---

Use this skill when the task needs current web information, cited external sources, or targeted lookup through the `tavily_search` tool.

Do not use Tavily for:
- repo-local facts already available in the workspace
- stable common knowledge that does not require web verification
- simple transformations, math, or reasoning with no external lookup

## Tool Summary

`tavily_search` takes:
- `query` required
- `max_results` from 1 to 20
- `topic` as `general` or `news`
- `search_depth` as `basic` or `advanced`
- `include_answer` boolean
- `include_raw_content` boolean
- `include_domains` optional allowlist
- `exclude_domains` optional denylist

The tool returns:
- a human-readable summary with an optional synthesized answer and numbered results
- structured data including `answer`, `response_time`, and `results[]`
- per-result fields such as `title`, `url`, `content`, `raw_content`, `published_date`, and `score`

Treat Tavily's synthesized answer as a shortcut, not as ground truth. Ground claims in the returned sources.

## Search Workflow

1. Define the information need before searching.
   Decide whether you need:
   - one authoritative fact
   - recent news
   - a comparison across multiple sources
   - confirmation or contradiction of a claim

2. Start with one focused query.
   Prefer short, concrete noun phrases over chatty prompts.
   Good:
   - `usd rub exchange rate today central bank russia`
   - `openai responses api background mode docs`
   - `site:docs.python.org asyncio timeout`
   Bad:
   - `can you please search the internet and tell me everything about...`

3. Pick the right retrieval mode.
   - Use `topic="news"` for recent events, announcements, incidents, launches, funding, policy changes, and time-sensitive data.
   - Use `topic="general"` for documentation, evergreen facts, company pages, and technical references.
   - Use `search_depth="basic"` for quick fact lookup or when you already know the likely source.
   - Use `search_depth="advanced"` for broad research, comparison, or when the first search is noisy.

4. Keep result count tight.
   - Default to `max_results=3` to `5`.
   - Use `6` to `8` for comparisons or conflicting claims.
   - Avoid `20` unless doing landscape research; it increases noise and synthesis cost.

5. Use domain filters deliberately.
   - `include_domains` when you want official docs, a vendor site, a standards body, or a specific news outlet.
   - `exclude_domains` when a domain dominates results with low-signal SEO pages or mirrors.
   - Prefer domain filtering over bloating the query with many qualifiers.

6. Use raw content sparingly.
   - Keep `include_raw_content=false` by default.
   - Turn it on only when snippets are insufficient and you need denser source text for extraction, quoting, or nuanced comparison.
   - Raw content enlarges tool output and should be reserved for second-pass lookup.

7. Refine instead of repeating the same search.
   If the first call is weak:
   - narrow the entity name
   - add the concrete date or version
   - switch `general` to `news`
   - constrain to official domains
   - split one broad question into two smaller searches

## Query Patterns

### Current events

Use:
- `topic="news"`
- `search_depth="advanced"` if signal is mixed
- `include_answer=true`
- `max_results=5`

Example:

```json
{
  "query": "gemini 3.1 flash lite preview thought signatures March 2026",
  "topic": "news",
  "search_depth": "advanced",
  "max_results": 5,
  "include_answer": true
}
```

### Official documentation lookup

Use:
- `topic="general"`
- `search_depth="basic"`
- `include_domains` for official docs
- `include_raw_content=false` on first pass

Example:

```json
{
  "query": "thought signatures Gemini API tool calling",
  "topic": "general",
  "search_depth": "basic",
  "max_results": 3,
  "include_domains": ["ai.google.dev"],
  "include_answer": true
}
```

### Verification of a claim

Run two passes:
- first on the likely primary source
- second on broader web coverage if the primary source is unclear or missing

Prefer exact entities, dates, and version numbers in the query.

### Comparison research

Use:
- `topic="general"`
- `search_depth="advanced"`
- `max_results=6` to `8`
- `include_answer=true`

Then compare the returned `results[]` by source quality, freshness, and agreement instead of trusting the synthesized answer alone.

## Source Handling Rules

- Cite the returned URLs or titles, not just "Tavily says".
- Prefer official docs, primary announcements, standards, or maintainers over reposts and listicles.
- If results conflict, say so explicitly and explain which source is more credible.
- For time-sensitive topics, mention the publication date when Tavily provides it.
- If the answer is not well supported by the returned results, run a follow-up search instead of overconfident synthesis.

## Practical Heuristics

- One precise search is better than one bloated search.
- Two narrow searches are often better than one broad search.
- Use `include_answer=true` for speed, but still inspect the result list.
- Use `include_raw_content=true` only when snippets are not enough.
- Reuse good domains from one search as `include_domains` in the next search.
- If you already have the exact vendor, spec, or organization, constrain to it immediately.

## Output Contract

After using `tavily_search`:
- answer the user's question directly
- include the most relevant sources
- mention uncertainty or conflicts when present
- distinguish between confirmed facts and likely inferences

If the search failed or returned weak evidence, say what was missing and what follow-up query would be needed.
