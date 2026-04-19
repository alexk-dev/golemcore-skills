---
name: brain
description: Use the golemcore_brain tool effectively for Brain wiki spaces, search status, hybrid search, intellisearch, assets, reindexing, mutating page operations, section-level patching, atomic transactions, link graph, and top-read tracking.
available: true
model_tier: smart
---

Use this skill when the task needs information from GolemCore Brain wiki content or the user explicitly asks you to create, update, patch, move, copy, delete, reindex, or inspect Brain pages and spaces through the `golemcore_brain` tool.

Do not use Brain for:
- repo-local facts already present in the workspace
- general web research that should come from public sources
- mutating operations unless the user explicitly asks for the specific Brain change

## Tool Summary

`golemcore_brain` takes:
- `operation` required
- `space_slug` optional Brain space slug; defaults to plugin settings
- `query` for `search_pages` and as optional retrieval text for `intellisearch`
- `search_mode` optional for `search_pages`: defaults to `auto`; use `fts` for exact full-text retrieval or `hybrid` for embedding-aware retrieval
- `context` for natural-language `intellisearch`
- `limit` optional result/page count, clamped from 1 to 20
- `path` Brain page path for page-specific operations
- `parent_path` parent section path for `create_page`
- `title`, `slug`, `content`, and `kind` for create/update operations
- `tags`, `summary` optional frontmatter metadata for `create_page` and `update_page`
- `revision` optional optimistic-lock revision for `update_page`
- `target_title` for `ensure_page`
- `target_parent_path`, `target_slug`, and `before_slug` for move/copy operations
- `dynamic_api_slug` optional Brain dynamic API slug for `intellisearch`
- `use_dynamic_endpoint` boolean; set `false` to force fallback search even when a dynamic endpoint is configured
- `patch_operation` (`APPEND`, `PREPEND`, or `REPLACE_SECTION`) and `expected_revision` for `patch_page`; `heading` for `REPLACE_SECTION`
- `operations` array for `wiki_tx` (each item: `{op: CREATE|UPDATE|DELETE, path, parentPath, slug, title, content, kind, expectedRevision}`)

Available operations:
- `list_spaces`
- `list_tree`
- `search_pages`
- `get_search_status`
- `intellisearch`
- `read_page`
- `create_page`
- `update_page`
- `patch_page`
- `delete_page`
- `ensure_page`
- `move_page`
- `copy_page`
- `list_assets`
- `reindex_space`
- `reindex_all_spaces`
- `get_wiki_graph`
- `wiki_top_accessed`
- `wiki_tx`

Mutating operations are gated by plugin settings. `create_page`, `update_page`, `patch_page`, `delete_page`, `ensure_page`, `move_page`, `copy_page`, `wiki_tx`, `reindex_space`, and `reindex_all_spaces` will fail unless Brain mutating operations are enabled.

## Workflow

1. Identify the Brain space and target content.
   Use the configured default space when the user does not name a space. Use `list_spaces` only when the space is unclear, and `list_tree` when the task depends on the page hierarchy.
2. Choose the narrowest read operation.
   Use `search_pages` for keyword, title, or concept lookup, `get_search_status` when index readiness or embedding readiness matters, `intellisearch` for relevance-based retrieval from a natural-language context, `read_page` when you know the page path, and `list_assets` when the user asks about files attached to a page.
3. Use `intellisearch` deliberately.
   Provide `context` for broad intent such as "find rollback guidance for a failed deploy". Provide `query` when there is a concise search phrase. Add `dynamic_api_slug` only when the user or environment identifies a specific Brain dynamic endpoint. Set `use_dynamic_endpoint=false` when you need the fallback path of Brain hybrid search plus page reads.
4. Pick the search mode intentionally.
   Omit `search_mode` or use `search_mode="auto"` when you want Brain to choose the default retrieval path. Use `search_mode="fts"` for exact keyword, wildcard, or title lookup. Use `search_mode="hybrid"` when conceptual relevance matters and Brain embeddings are expected to be configured. Do not use removed aliases such as `semantic` or `lexical`.
5. Inspect before mutating.
   For updates, deletes, moves, and copies, read the current page first unless the user already provided the exact current state. Preserve and pass `revision` when available for safer updates.
6. Keep mutating operations explicit.
   Before calling a mutating operation, verify the target space, path, and intended change. If the user's request is ambiguous, ask for the missing path, destination, or content rather than guessing.
7. Queue reindex only on explicit intent.
   Use `reindex_space` for one space and `reindex_all_spaces` only when the user asks to rebuild all Brain search indexes or after a bulk import/change that needs a full refresh.
8. Report the Brain result in practical terms.
   Summarize page paths, titles, and the outcome. Mention when the tool is disabled, the base URL is missing, mutating operations are disabled, or the Brain API rejected the request.

## Operation Patterns

### Search and read

Use `search_pages` when the user gives concrete terms:

```json
{
  "operation": "search_pages",
  "space_slug": "docs",
  "query": "deployment rollback",
  "search_mode": "fts",
  "limit": 5
}
```

Then use `read_page` on the most relevant `path`:

```json
{
  "operation": "read_page",
  "space_slug": "docs",
  "path": "ops/deployment/rollback"
}
```

Use hybrid search when the user asks for conceptually related content:

```json
{
  "operation": "search_pages",
  "space_slug": "docs",
  "query": "how do we recover after a failed rollout",
  "search_mode": "hybrid",
  "limit": 5
}
```

### Search status

Use `get_search_status` when the user asks whether search is current, whether embeddings are ready, why hybrid search may fall back to FTS, or whether a reindex may be needed:

```json
{
  "operation": "get_search_status",
  "space_slug": "docs"
}
```

The result includes fields such as `ready`, `indexedDocuments`, `fullTextIndexedDocuments`, `embeddingDocuments`, `embeddingIndexedDocuments`, `staleDocuments`, `embeddingsReady`, `embeddingModelId`, `lastFullRebuildAt`, `lastUpdatedAt`, and `lastIndexingError`.

### Relevance lookup

Use `intellisearch` when the user asks for the best matching Brain material for a broader goal:

```json
{
  "operation": "intellisearch",
  "space_slug": "docs",
  "context": "Need guidance for rolling back a failed production deployment",
  "query": "rollback failed deploy",
  "limit": 5
}
```

### Reindex

Use `reindex_space` when the user asks to rebuild search for one Brain space:

```json
{
  "operation": "reindex_space",
  "space_slug": "docs"
}
```

Use `reindex_all_spaces` only for explicit full-index rebuild requests:

```json
{
  "operation": "reindex_all_spaces"
}
```

### Create page

Use `create_page` only for explicit create requests:

```json
{
  "operation": "create_page",
  "space_slug": "docs",
  "parent_path": "ops/deployment",
  "title": "Rollback Checklist",
  "slug": "rollback-checklist",
  "content": "# Rollback Checklist\n\n...",
  "kind": "PAGE"
}
```

### Update page

Use `update_page` after reading the current page and preparing the full replacement content:

```json
{
  "operation": "update_page",
  "space_slug": "docs",
  "path": "ops/deployment/rollback-checklist",
  "title": "Rollback Checklist",
  "content": "# Rollback Checklist\n\n...",
  "revision": "known-revision-if-available"
}
```

### Move or copy page

Use `move_page` or `copy_page` only after verifying the source path and target parent:

```json
{
  "operation": "move_page",
  "space_slug": "docs",
  "path": "ops/old-runbook",
  "target_parent_path": "ops/deployment",
  "target_slug": "rollback-runbook"
}
```

### Patch page

Use `patch_page` for section-level edits that avoid rewriting the full page body. The operation rejects with HTTP 409 when `expected_revision` is stale, so always read the page first and pass its current revision.

```json
{
  "operation": "patch_page",
  "space_slug": "docs",
  "path": "ops/deployment/rollback-checklist",
  "patch_operation": "APPEND",
  "expected_revision": "known-revision-from-read_page",
  "content": "\n\n## Post-rollback\n\n- Notify #ops channel"
}
```

Use `REPLACE_SECTION` with `heading` to rewrite the body under a heading. The matched heading line is preserved — pass only the new body in `content` (no leading `##`), otherwise the heading will be duplicated. The replaced range runs up to the next heading of equal or higher level.

```json
{
  "operation": "patch_page",
  "path": "ops/deployment/rollback-checklist",
  "patch_operation": "REPLACE_SECTION",
  "heading": "Post-rollback",
  "expected_revision": "known-revision-from-read_page",
  "content": "\nRewritten steps...\n"
}
```

### Transactional batch edits

Use `wiki_tx` when several related edits must be applied together.

Pre-validation is best-effort: before persisting anything, Brain checks that every `UPDATE`/`DELETE` path exists, every `CREATE` has a non-blank title, and every `UPDATE`'s `expectedRevision` matches the current revision. If any pre-check fails, the batch is rejected and nothing is persisted.

Once pre-validation passes, operations are applied sequentially. A failure mid-batch (for example a filesystem I/O error, or a slug collision discovered during `CREATE`) leaves earlier operations persisted. Write batches so ops are safe to retry, and re-read affected paths if a mid-batch failure is reported.

`CREATE` requires `parentPath`, `title`, `slug`, `kind`, `content`. `UPDATE` requires `path`, `title`, `slug`, `content`, `expectedRevision`. `DELETE` requires `path`.

### Choosing between `update_page`, `patch_page`, and `wiki_tx`

- Use `update_page` for a single page when you need to rewrite the full body or change title/slug/frontmatter. Pass `revision` for optimistic concurrency.
- Use `patch_page` for a single page when you only need to append, prepend, or rewrite one section. Pass `expected_revision`. Avoids sending the full body and reduces the risk of overwriting unrelated edits.
- Use `wiki_tx` when several related pages must change together (create + update, multi-page rename, bulk delete). Pre-validation rejects stale revisions or missing paths before any write.

```json
{
  "operation": "wiki_tx",
  "space_slug": "docs",
  "operations": [
    {"op": "CREATE", "parentPath": "ops", "title": "Runbook index", "slug": "index", "kind": "SECTION", "content": ""},
    {"op": "UPDATE", "path": "ops/rollback", "title": "Rollback", "slug": "rollback", "content": "...", "expectedRevision": "rev-123"}
  ]
}
```

### Wiki graph

Use `get_wiki_graph` to find orphan pages (not linked from anywhere) and dangling outgoing links across the whole space. Useful before a documentation cleanup pass.

```json
{"operation": "get_wiki_graph", "space_slug": "docs"}
```

### Top accessed pages

Use `wiki_top_accessed` to list the most-read pages in a space. Read counts are recorded automatically on each `read_page` call.

```json
{"operation": "wiki_top_accessed", "space_slug": "docs", "limit": 10}
```

### Page metadata

Pass `tags` and `summary` to `create_page` or `update_page` to store YAML frontmatter on the page:

```json
{
  "operation": "create_page",
  "parent_path": "ops",
  "title": "Rollback Checklist",
  "slug": "rollback-checklist",
  "tags": ["ops", "runbook"],
  "summary": "How to recover a failed production rollout.",
  "content": "# Rollback Checklist\n\n..."
}
```

## Safety Rules

- Treat Brain content as user-owned documentation. Do not make destructive or restructuring changes without explicit user intent.
- Do not call `delete_page` unless the user specifically asked to delete that path.
- Do not call `reindex_all_spaces` unless the user asked for a full rebuild or confirmed that a broad rebuild is acceptable.
- Do not overwrite page content from memory. Read the page first or use content the user supplied in the current conversation.
- If mutating operations are disabled, report that settings blocked the mutation instead of retrying with a different mutating operation.
- If a page path, destination, or space is ambiguous, ask a focused question before mutating.

## Output Contract

After using `golemcore_brain`:
- answer the user's question or state the completed change directly
- include relevant Brain space and page paths
- for search results, mention the practical retrieval path when Brain returns one, such as `fts`, `hybrid`, `fts-fallback`, or `empty-query`
- distinguish retrieved Brain facts from your own inference
- mention disabled tool/settings or API errors when they affect the result
- for mutating operations, state exactly what changed and whether verification readback was performed
