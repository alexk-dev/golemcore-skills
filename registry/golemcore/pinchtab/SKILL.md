---
name: pinchtab
description: Use PinchTab browser tools to navigate websites, inspect dynamic pages, find interactive elements, perform browser actions, extract readable text, and capture screenshots efficiently.
available: true
model_tier: smart
---

Use this skill when the task requires real browser interaction through PinchTab, especially for dynamic websites, multi-step page flows, form submission, DOM inspection, or visual verification.

## Workflow

1. Establish browser health only when needed.
   If PinchTab fails, seems unavailable, or the task depends on a long browser flow, start with `pinchtab_health`. Use `pinchtab_instances` or `pinchtab_tabs` only when you need to inspect, start, stop, or recover orchestrator state.
2. Navigate first and keep the returned `tabId`.
   Use `pinchtab_navigate` to open the target page. Reuse the returned `tabId` across follow-up tools so the browser state stays stable.
3. Prefer snapshots for page understanding.
   Use `pinchtab_snapshot` with the default compact interactive format to get reusable refs like `e5` or `e12`. This is usually the most token-efficient way to understand the page structure.
4. Refresh refs after page-changing actions.
   After navigation, click, fill, select, or any action that may change the DOM, take a fresh snapshot before acting again. Do not trust stale refs.
5. Use semantic find when the target is unclear.
   If you know what you want but not the ref, use `pinchtab_find` with a natural-language query, then pass `best_ref` into `pinchtab_action`.
6. Use actions deliberately.
   Prefer `ref` from snapshot or find. Use `selector` only when a stable ref is unavailable. For text entry, prefer `fill` for replacing content and `type` for incremental typing.
7. Use text extraction for reading-heavy tasks.
   When the goal is to read an article, extract page content, or summarize text, use `pinchtab_text` instead of repeated snapshots or screenshots.
8. Use screenshots sparingly.
   `pinchtab_screenshot` is for visual confirmation, layout issues, or user-facing evidence. It is not the default tool for reading page content.

## Tool Selection

- `pinchtab_navigate`: open or reuse a page
- `pinchtab_snapshot`: inspect the current DOM and capture refs
- `pinchtab_find`: locate an element semantically
- `pinchtab_action`: click, fill, type, select, hover, press, focus, or scroll
- `pinchtab_text`: extract readable text
- `pinchtab_screenshot`: capture visual evidence
- `pinchtab_tabs`: inspect or close tabs
- `pinchtab_instances`: inspect, start, or stop browser instances
- `pinchtab_health`: verify the PinchTab server

## Efficiency Rules

- Default to one active tab unless the task truly needs parallel page state.
- Prefer compact interactive snapshots over full snapshots.
- Prefer `diff=true` snapshots only after you already have a stable baseline and need to inspect what changed.
- Prefer readable text extraction over screenshots for textual answers.
- Do not repeatedly list instances or tabs unless state recovery is part of the task.

## PinchTab-Specific Cautions

- `pinchtab_navigate` with `instance_id` but without `tab_id` opens a fresh tab and does not support the advanced navigate flags. If you need those flags, open the tab first, then navigate again with `tab_id`.
- `pinchtab_action` needs a valid target for most interactive operations. For `click`, `focus`, and `hover`, provide a `ref` or `selector`. For `fill` and `type`, also provide text.
- Refs are page-state dependent. If the DOM changed, reacquire refs before the next action.

## Output Contract

When reporting back to the user:

- state what page or step you reached
- summarize what actions were performed
- mention blockers such as missing auth, broken page state, or unavailable PinchTab service
- include a screenshot only when it materially helps explain the result
