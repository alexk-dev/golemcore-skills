---
name: brain
description: Use the golemcore_brain tool effectively for Brain wiki spaces, page lookup, intellisearch, assets, and write-gated page operations.
available: true
model_tier: smart
---

Use this skill when the task needs information from GolemCore Brain wiki content or the user explicitly asks you to create, update, move, copy, or delete Brain pages through the `golemcore_brain` tool.

Do not use Brain for:
- repo-local facts already present in the workspace
- general web research that should come from public sources
- write operations unless the user explicitly asks for the specific Brain change

## Tool Summary

`golemcore_brain` takes:
- `operation` required
- `space_slug` optional Brain space slug; defaults to plugin settings
- `query` for `search_pages` and as optional retrieval text for `intellisearch`
- `context` for natural-language `intellisearch`
- `limit` optional result/page count, clamped from 1 to 20
- `path` Brain page path for page-specific operations
- `parent_path` parent section path for `create_page`
- `title`, `slug`, `content`, and `kind` for create/update operations
- `revision` optional optimistic-lock revision for `update_page`
- `target_title` for `ensure_page`
- `target_parent_path`, `target_slug`, and `before_slug` for move/copy operations
- `dynamic_api_slug` optional Brain dynamic API slug for `intellisearch`
- `use_dynamic_endpoint` boolean; set `false` to force fallback search even when a dynamic endpoint is configured

Available operations:
- `list_spaces`
- `list_tree`
- `search_pages`
- `intellisearch`
- `read_page`
- `create_page`
- `update_page`
- `delete_page`
- `ensure_page`
- `move_page`
- `copy_page`
- `list_assets`

Write operations are gated by plugin settings. `create_page`, `update_page`, `delete_page`, `ensure_page`, `move_page`, and `copy_page` will fail unless Brain write operations are enabled.

## Workflow

1. Identify the Brain space and target content.
   Use the configured default space when the user does not name a space. Use `list_spaces` only when the space is unclear, and `list_tree` when the task depends on the page hierarchy.
2. Choose the narrowest read operation.
   Use `search_pages` for keyword or title lookup, `intellisearch` for relevance-based retrieval from a natural-language context, `read_page` when you know the page path, and `list_assets` when the user asks about files attached to a page.
3. Use `intellisearch` deliberately.
   Provide `context` for broad intent such as "find rollback guidance for a failed deploy". Provide `query` when there is a concise search phrase. Add `dynamic_api_slug` only when the user or environment identifies a specific Brain dynamic endpoint. Set `use_dynamic_endpoint=false` when you need the deterministic fallback path of Brain search plus page reads.
4. Inspect before mutating.
   For updates, deletes, moves, and copies, read the current page first unless the user already provided the exact current state. Preserve and pass `revision` when available for safer updates.
5. Keep write operations explicit.
   Before calling a write operation, verify the target space, path, and intended change. If the user's request is ambiguous, ask for the missing path, destination, or content rather than guessing.
6. Report the Brain result in practical terms.
   Summarize page paths, titles, and the outcome. Mention when the tool is disabled, the base URL is missing, writes are disabled, or the Brain API rejected the request.

## Operation Patterns

### Search and read

Use `search_pages` when the user gives concrete terms:

```json
{
  "operation": "search_pages",
  "space_slug": "docs",
  "query": "deployment rollback",
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

## Safety Rules

- Treat Brain content as user-owned documentation. Do not make destructive or restructuring changes without explicit user intent.
- Do not call `delete_page` unless the user specifically asked to delete that path.
- Do not overwrite page content from memory. Read the page first or use content the user supplied in the current conversation.
- If write operations are disabled, report that settings blocked the mutation instead of retrying with a different write operation.
- If a page path, destination, or space is ambiguous, ask a focused question before mutating.

## Output Contract

After using `golemcore_brain`:
- answer the user's question or state the completed change directly
- include relevant Brain space and page paths
- distinguish retrieved Brain facts from your own inference
- mention disabled tool/settings or API errors when they affect the result
- for write operations, state exactly what changed and whether verification readback was performed
