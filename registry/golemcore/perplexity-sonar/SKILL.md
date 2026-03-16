---
name: perplexity-sonar
description: Use the perplexity_ask tool effectively for grounded search-backed answers, cited research, and focused follow-up exploration.
available: true
model_tier: smart
---

Use this skill when the task needs a grounded answer with sources, search-backed synthesis, or structured follow-up exploration through the `perplexity_ask` tool.

Do not use Perplexity Sonar for:
- repo-local questions that can be answered from the workspace
- simple factual lookups that do not justify a full answer-generation call
- tasks where you need raw page extraction instead of search-backed synthesis

## Tool Summary

`perplexity_ask` takes:
- `question` required
- `model` optional: `sonar`, `sonar-pro`, `sonar-deep-research`, or `sonar-reasoning-pro`
- `system_prompt` optional
- `search_mode` optional: `web`, `academic`, or `sec`
- `search_domain_filter` optional domain allowlist
- `search_language_filter` optional language list such as `en`
- `search_recency_filter` optional: `hour`, `day`, `week`, `month`, or `year`
- `max_tokens` optional
- `temperature` optional
- `return_related_questions` optional boolean
- `return_images` optional boolean
- `reasoning_effort` optional: `minimal`, `low`, `medium`, or `high`

The tool returns:
- a synthesized answer
- `search_results[]` with source titles, URLs, dates, and snippets
- optional `related_questions`
- optional `images`
- token `usage`

Treat the synthesized answer as useful compression, not as unquestionable truth. Ground key claims in the returned sources.

## Search Workflow

1. Define the actual information need.
   Decide whether you need:
   - a fast grounded answer
   - academic evidence
   - SEC or filing-based research
   - a deeper comparison across multiple sources

2. Pick the right model.
   - Use `sonar` for quick, lower-cost grounded answers.
   - Use `sonar-pro` when answer quality matters more than speed.
   - Use `sonar-deep-research` for broader research and evidence gathering.
   - Use `sonar-reasoning-pro` when the question needs harder reasoning on top of retrieved evidence.

3. Pick the right search mode.
   - `web` for normal web research, product docs, vendor pages, and news-like lookup.
   - `academic` for papers, scholarly sources, and research-backed questions.
   - `sec` for public-company filings and investor relations research.

4. Shape the retrieval space.
   - Use `search_domain_filter` when you already know the authoritative domains.
   - Use `search_language_filter` when the answer should come from a specific language corpus.
   - Use `search_recency_filter` for time-sensitive topics.

5. Keep the question focused.
   Ask one well-scoped question at a time.
   Better:
   - `What changed in Google's Gemini thought signature requirements for tool calling in 2026?`
   - `What do the latest Microsoft 10-K filings say about Copilot revenue risk factors?`
   Worse:
   - `Tell me everything about Gemini and AI search and compare all providers`

6. Use answer-shaping parameters deliberately.
   - Keep `temperature` low for factual work.
   - Use `system_prompt` for output shape or constraints, not to overload the question.
   - Turn on `return_related_questions` when exploring a topic; turn it off when you need a concise one-shot answer.
   - Keep `return_images=false` unless images materially help.
   - Increase `reasoning_effort` only when the task is genuinely complex.

7. Inspect sources before finalizing.
   Read the returned `search_results[]` and verify that the sources are actually authoritative and current enough.

## Query Patterns

### General grounded answer

```json
{
  "question": "What changed in Gemini API thought signature handling for tool calls in 2026?",
  "model": "sonar-pro",
  "search_mode": "web",
  "search_domain_filter": ["ai.google.dev"],
  "search_recency_filter": "year",
  "return_related_questions": false,
  "reasoning_effort": "medium"
}
```

### Academic research

```json
{
  "question": "What recent papers compare retrieval augmentation strategies for code assistants?",
  "model": "sonar-deep-research",
  "search_mode": "academic",
  "search_language_filter": ["en"],
  "return_related_questions": true
}
```

### SEC or filings research

```json
{
  "question": "What risk factors related to AI are highlighted in Microsoft's latest 10-K filing?",
  "model": "sonar-pro",
  "search_mode": "sec",
  "return_related_questions": false
}
```

## Source Handling Rules

- Cite the returned source URLs or titles, not just "Perplexity says".
- Prefer primary sources and authoritative domains over commentary.
- If Perplexity's answer and the source snippets feel misaligned, trust the sources and refine the question.
- For time-sensitive answers, mention recency and the dates provided by the sources.
- If the returned sources are weak, rerun with stronger domain or recency constraints rather than overconfident synthesis.

## Practical Heuristics

- Ask one concrete question, not a stack of questions.
- Use `search_domain_filter` whenever the authoritative source is known.
- Use `academic` and `sec` modes only when they match the evidence you actually need.
- Prefer `sonar` or `sonar-pro` first; escalate to deeper or more reasoning-heavy models when necessary.
- Related questions are for exploration, not for padding the final answer.

## Output Contract

After using `perplexity_ask`:
- answer the user's question directly
- cite the best supporting sources
- separate confirmed facts from likely inferences
- mention conflicts, uncertainty, or missing evidence when present
- propose the next best follow-up question if the result set is incomplete
