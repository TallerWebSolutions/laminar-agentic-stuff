---
name: laminar-mcp
description: Guides Laminar MCP sessions for demand creation, story map operations, source context loading, and handoff ADRs. Use when working on Laminar demands via the remote Laminar MCP and you see wrong or empty client/product scope, plans from source-context lists without per-id loads, needless raw transcripts, same-step or same-release story-map peers, anchored ADR conflicts, MCP transitions/assignments, broken or silent MCP, or mentions of Laminar MCP, demands, work item custom IDs, story map, anchored or source context, or Laminar handoff.
---

# Laminar MCP

## Overview

Server `tools/list` wins on names and args over this file. Assume **only** MCP access — not a checkout, workspace, or app shell. Prefer **signals** from `load_source_context`; stop loading when the next step is clear.

## Quick start

`get_current_context` → if needed `list_clients` / `list_products` → `set_context` (clientId, then productId).
Lists need **client**; story map and releases need **product**. `clear_context` only to drop product on purpose.

→ Full tool catalog: [references/tool-catalog.md](references/tool-catalog.md)

## Create demand

Follow in order — do not skip.

**1. Source context gate** *(mandatory — no planning or codebase work until done)*
`list_source_contexts` → identify relevant IDs → `load_source_context` on each

**2. Duplicate check**
`list_work_items` (open) → if an existing item covers the same problem, update it instead of creating

**3. Story map context**
`get_story_map` → `list_story_map_activities` + `list_story_map_steps` + `list_story_map_work_items`
Use peer items and existing steps as input to the brainstorm (step 4), not just for placement

**4. Solution brainstorm**
Invoke `/grill-me` to stress-test scope before drafting.
If not installed: question the minimal fix, explore broader alternatives, consider story map journey fit.
*(Recommend installing: `.agents/skills/grill-me/`)*

**5. Draft** — apply conventions below

**6. Create** → `confirmed: false` → echo summary → approved → `confirmed: true`

**7. Story map placement** *(always ask, even if not mentioned by user)*
"Should I place this on a story map step?" → `move_work_item_to_step` if yes

**8. Release assignment** *(always ask, even if not mentioned by user)*
"Should I assign this to a release?" → `assign_work_items_to_release_batch` if yes

→ Write contracts & batch ops: [references/making-changes.md](references/making-changes.md)

## Read existing demand

If you already have the ID: `get_work_item` → `get_work_item_anchored_context` → `list_source_contexts` → `load_source_context` (relevant ids) → `get_story_map`

If you don't have the ID: `list_work_items` first to find the `customId`, then follow the flow above. **Never pass free-text to `get_work_item`** — it only accepts a `customId` (e.g. `TAL-131`) or internal `workItemId`.

Synthesize intent from work item + anchored context; ground details in loaded source contexts.

## Demand conventions

**Language**: Match the conversation's language by default. At `confirmed: false` preview, ask: *"Translate to [language] before creating?"*

**Title**:
- Features: actor + action ("Admin creates team to set up board")
- Bugs: what the user can't do or the observable broken behavior ("Demand creation fails with 'No statuses found' in new orgs")
- Never prefix with type (`Bug:`, `Feat:`) — type is a structured field

**Acceptance criteria**: Gherkin recommended — Feature / As a / I want / So that + Background / Scenario / Given / When / Then

**Bug description structure**: Problem → Root cause (if known) → Steps to reproduce → Acceptance criteria (Gherkin)

## Pitfalls

| Symptom | Fix |
|---------|-----|
| Jumped to planning before source contexts | Hard gate: `list_source_contexts` + `load_source_context` FIRST |
| No duplicate check | `list_work_items` before every create |
| Story map / release skipped after creation | Both are mandatory post-create checkpoints — always ask |
| Minimal fix without exploring scope | `/grill-me` or inline brainstorm before drafting |
| Write executed without approval | `confirmed: false` → echo summary → `confirmed: true` only after user accepts |
| Story map / releases errors | Set clientId + productId in context first |
| `get_valid_transitions` with only a customId | Load `workItemId` via `get_work_item` first |
| Batch previews assumed complete | ACL + row validation happen on `confirmed: true`; inspect `failures` after execute |
| `pendingOperationId` reused across calls | It's informational — re-call the **same tool with same args** to execute, not the pending id |
| Raw source context dumps by default | Signals first; `includeRawText: true` only if signals are missing or insufficient |
| Chat-only "done" on handoff | `put_work_item_anchored_context` + `expectedVersion` — chat summary alone is not persisted |
| Wrong reorder API | `reorder_story_map_step` / `_activity` take `targetPosition` (integer); `direction` is only for `reorder_story_map_release` |
