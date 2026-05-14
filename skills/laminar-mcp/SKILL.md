---
name: laminar-mcp
description: Use when working on Laminar demands via the remote Laminar MCP and you see wrong or empty client/product scope, plans from source-context lists without per-id loads, needless raw transcripts, same-step or same-release story-map peers, anchored ADR conflicts, MCP transitions/assignments, broken or silent MCP, or mentions of Laminar MCP, demands, TAL-* ids, story map, anchored or source context, or Laminar handoff.
---

# Laminar MCP

## Overview

**Laminar** is the product behind this MCP. **Server `tools/list` (or equivalent)** wins on names and args over this file. Assume **only** MCP access—not a checkout, workspace, or app shell. Prefer **signals** from `load_source_context`; stop loading when the next step is clear.

## MCP tool catalog

Rough registration order; always prefer live **inputSchema** from the server.

| Area | Tools |
|------|--------|
| **Session** | `get_current_context`, `set_context` (`clientId`?, `productId`?), `clear_context` |
| **Source context** | `list_source_contexts` (`limit`?), `load_source_context` (`sourceContextIds`, `includeRawText`?) |
| **Portfolio / org lists** | `list_clients`, `list_products` (`clientId`? or session client), `list_statuses`, `list_team_members`, `list_blocker_types`, `list_releases` (session `productId` or `productId` arg) |
| **Work items (read)** | `list_work_items` (`completionFilter`?: `open` default \| `all`), `get_work_item` (`query`, `descriptionFormat`?: `plain` \| `json` \| `both`), `get_valid_transitions` (`workItemId` — id string from reads) |
| **Story map (read)** | `get_story_map` (`productId`?), `list_story_map_activities` (`storyMapId`), `list_story_map_steps` (`storyMapId`), `list_story_map_releases` (`storyMapId` — order of releases on the map strip), `list_story_map_work_items` (`productId`?) |
| **Work items (write, `confirmed`)** | `create_work_item`, `transition_work_item`, `assign_work_item`, `unassign_work_item`, `update_work_item`, `add_blocker`, `remove_blocker` |
| **Story map (write, `confirmed`)** | `create_story_map_activity`, `update_story_map_activity`, `delete_story_map_activity`, `create_story_map_step`, `update_story_map_step`, `delete_story_map_step`, `move_step_to_activity`, `move_work_item_to_step`, `reorder_story_map_step`, `reorder_story_map_activity`, `reorder_story_map_release` |
| **Batch writes (`confirmed`)** | **`create_story_map_activities_batch`**, **`create_story_map_steps_batch`**, **`create_work_items_batch`**, **`assign_work_items_to_release_batch`**, **`update_work_items_batch`** — up to **200** rows per call; see **Agent: batch operations** |
| **Anchored ADR** | `get_work_item_anchored_context` (`query`), `put_work_item_anchored_context` (`workItemQuery`, structured sections, `expectedVersion` — **not** `confirmed`) |

**ID and query conventions**

- **`query` / `workItemQuery`**: **`TAL-*`** or **`workItemId`**; used by `get_work_item`, `get_work_item_anchored_context`, `put_work_item_anchored_context`, `transition_work_item`, `assign_work_item`, `unassign_work_item`.
- **`workItemId`**: the **`workItemId`** value from **`get_work_item`** / **`list_work_items`** (not **`TAL-*`**); required by `update_work_item`, `add_blocker`, `remove_blocker`, `move_work_item_to_step`, `get_valid_transitions`.
- **Creates** return new ids in the execute response (e.g. `create_work_item` → `workItemId`, `customId`; story-map creates → `userActivityId` / `userStepId`; `add_blocker` → `blockerId`).

## For the collaborator (human)

- Attach **`@laminar-mcp`** when starting demand work so the agent reads this playbook.
- Give **demand id** (`TAL-*` or **`workItemId`**) and **client/product scope** when it must not be guessed; say if you want **anchored handoff** at session end.
- If MCP fails: confirm MCP is enabled + auth; retry or paste **`tools/list`** for the agent.

## Agent: session scope

`get_current_context` → if needed `list_clients` / `list_products` → `set_context` (`clientId`, then `productId`). Lists need **client**; story map and releases need **product**. `clear_context` only to drop product on purpose.

## Agent: demand context (order)

Demand record → anchored ADR → evidence → map → peers.

| Goal | Tool | Notes |
|------|------|--------|
| Demand fields | `get_work_item` | `query` = id. `descriptionFormat: "plain"` to read; `json`/`both` only if editing description. |
| Handoff doc | `get_work_item_anchored_context` | Version + markdown. |
| Pick docs | `list_source_contexts` | IDs + short lines only—not a planning basis. |
| Evidence | `load_source_context` | Per relevant id before substantive answers. Signals default; `includeRawText: true` only if signals missing or insufficient. |
| Map | `get_story_map` | Then `list_story_map_activities` / `list_story_map_steps` / `list_story_map_releases` (`storyMapId`), `list_story_map_work_items`, `list_releases` (product-scoped release list), `list_work_items` (`completionFilter` if needed). |

Synthesize intent from `get_work_item` + `get_work_item_anchored_context`; ground details in loaded source contexts.

## Agent: handoff

Use **`put_work_item_anchored_context`**: `get_work_item_anchored_context` → merge sections per **live server schema** (`decisions`, `constraints`, `openQuestions`, `stateItems`, `outOfScope`, `scenarios` + coverage enum, optional `documentTitle`) → put with `expectedVersion` (`0` if new) → on conflict, refetch and retry. Optional: `update_work_item` via TipTap JSON from `get_work_item` (`descriptionFormat: "json"`).

## Agent: writes — `confirmed` two-step contract

Every **confirmed** work-item and story-map write tool below requires a `confirmed: boolean` arg (**not** `put_work_item_anchored_context`; that path uses **`expectedVersion`**). For those tools the same call shape is used twice:

1. **Preview** — call with `confirmed: false`. Server returns `{ confirmed: false, summary, pendingOperationId, expiresAt }` and **does not mutate**. Echo `summary` to the user (or surface it in your plan); wait for explicit approval.
2. **Execute** — call with `confirmed: true`. Server re-checks ACL and runs the mutation. Returns `{ confirmed: true, ok: true, ... }` (e.g. `workItemId`/`customId` from `create_work_item`, `userActivityId`, `userStepId`, `blockerId`).

Tools covered by this contract: `create_work_item`, `update_work_item`, `transition_work_item`, `assign_work_item`, `unassign_work_item`, `add_blocker`, `remove_blocker`, `create_story_map_activity` / `update_story_map_activity` / `delete_story_map_activity`, `create_story_map_step` / `update_story_map_step` / `delete_story_map_step`, `move_work_item_to_step`, `move_step_to_activity`, `reorder_story_map_step`, `reorder_story_map_activity`, `reorder_story_map_release`, plus the **batch** tools in **Agent: batch operations**. **`put_work_item_anchored_context`** uses **`expectedVersion`** (and structured sections), not `confirmed`.

**Work-item write args (high signal)**

- **`create_work_item`**: `title`, `clientId`, `statusId`, `type`, `serviceClass`; optional `description` (TipTap JSON string), `productId`, `releaseId`, `userStepId` (requires `productId` on same product). Resolve ids with `list_clients`, `list_statuses`, `list_products`, `list_releases`, story-map list tools — no natural-language resolution.
- **`transition_work_item`**: `workItemQuery`, `toStatusId`. Clears active attributions; assign again with `assign_work_item` if needed. Call **`get_valid_transitions`** first (needs **`workItemId`** from reads).
- **`assign_work_item` / `unassign_work_item`**: `workItemQuery`, `teamMemberId` (from `list_team_members`).
- **`update_work_item`**: **`workItemId`**, optional `title`, `description` (full TipTap JSON), `type`, `serviceClass`, `productId`, `releaseId`.
- **`add_blocker`**: **`workItemId`**, `blockerTypeId` (from `list_blocker_types`), `reason` (≤ 600 chars); item must be in a **touch** status and have no active blocker.
- **`remove_blocker`**: **`workItemId`**; removes active blocker.

Never collapse the two steps into one — when planning a sequence, first run the entire preview chain with `confirmed: false`, present the plan, then re-run the same calls with `confirmed: true` after approval. If you need new IDs from an earlier step (e.g. the `userActivityId` returned by `create_story_map_activity`), execute that step before previewing the dependent step.

## Agent: story-map writes

Set product in `set_context`, then `get_story_map` → `list_story_map_activities` / `list_story_map_steps` / `list_story_map_releases` for current shape. Use IDs only (no name lookups).

| Goal | Tool |
|------|------|
| Add an activity / step | `create_story_map_activity` (returns `userActivityId`) → `create_story_map_step` (`storyMapId`, `userActivityId`, `name`; optional `position`) |
| Rename / delete | `update_story_map_activity` / `delete_story_map_activity`, `update_story_map_step` / `delete_story_map_step` |
| Reorder rows | `reorder_story_map_activity` (`userActivityId`, **`targetPosition`** 0-based in map). `reorder_story_map_step` (`userStepId`, **`targetPosition`** 0-based **within its activity** — use `list_story_map_steps` first). |
| Reorder release strip | `reorder_story_map_release` (`storyMapId`, `releaseId`, **`direction`**: `up` \| `down`) — call `list_story_map_releases` for current order |
| Move step between activities | `move_step_to_activity` (`userStepId`, `userActivityId`) — step appended under target activity |
| Place / clear a demand on the map | `move_work_item_to_step` (**`workItemId`**, `userStepId` or `null` to clear — does **not** change release) |
| Create a demand on the map | `create_work_item` with `productId` + optional `userStepId` (+ other required fields) |
| Many new activities / steps / demands | See **Agent: batch operations** (`create_story_map_activities_batch` → `create_story_map_steps_batch` → optional `create_work_items_batch`) |

## Agent: batch operations

Use batch tools when you need **many** similar changes and want **one** preview, **one** user approval, and **one** execute — no per-row narration.

| Tool | Role |
|------|------|
| `create_story_map_activities_batch` | `storyMapId` + `activities: [{ name }]`. Appends activities (no default steps). Execute returns `activities: [{ name, userActivityId }]`. |
| `create_story_map_steps_batch` | `storyMapId` + `steps: [{ name, userActivityId }]`. Global append positions (same model as repeated `create_story_map_step`). Returns `steps: [{ name, userStepId }]`. |
| `create_work_items_batch` | Shared **`clientId`, `statusId`, `type`, `serviceClass`** (ids from list tools); optional `productId`, `releaseId` (defaults for every row). `items: [{ title, description?, releaseId?, userStepId? }]`. If any row has `userStepId`, **shared `productId` is required**. Partial failure: response may include `failures: [{ title, error }]`. |
| `assign_work_items_to_release_batch` | **`releaseId`** (**id** **or `null`** to clear release on each listed row); **`workItemQueries`** (demand **`TAL-*`** or **`workItemId`** each). **Release-only** bulk — does not move story-map rows. When session **`productId`** is pinned, **`releaseId`** must belong to that product. Per-row **`failures`**. |
| `update_work_items_batch` | `items` rows with `workItemQuery` plus the same patch fields as `update_work_item` (`productId` / `releaseId` / `userStepId` may be `null` to clear). Prefer **`assign_work_items_to_release_batch`** for release-only bulk. Per-row **`failures`**. |

**Order for a story-map import**

1. `create_story_map_activities_batch` (execute) → collect `userActivityId`s.  
2. `create_story_map_steps_batch` (execute) → collect `userStepId`s if you need placements.  
3. `create_work_items_batch` with shared `productId` and per-item `userStepId` where needed.

**Batch arguments:** Use **ids returned** by **`list_*`** tools and story-map reads for shared fields (**`clientId`**, **`statusId`**, **`productId`**, **`releaseId`**, **`storyMapId`**, step/activity ids). Where a parameter is **`workItemQueries`**, each entry is **`TAL-*`** or **`workItemId`** — same as **`workItemQuery`** on single-row tools. The server **does not** resolve names or free text into ids.

**Session filters**

When `set_context` has pinned `clientId` and/or `productId`, bulk **assign/update** rejects rows outside that portfolio (mirror single-tool behavior).

## Agent: story-map peers

With product in `set_context`: `get_story_map` → `list_story_map_work_items` → find row → filter others by same `userStepId`, `userActivityName`, or `releaseId`/`releaseName` → selectively `get_work_item` / anchored / `load_source_context`.

## Supporting reads

Before writes: **`get_valid_transitions`** (**`workItemId`** from reads) before **`transition_work_item`**; **`list_blocker_types`** before **`add_blocker`**; **`list_team_members`** before assign/unassign. **`list_work_items`** defaults to **`open`**; pass **`completionFilter: "all"`** when you need done items too. Prefer reads first; always preview writes with `confirmed: false` before executing.

**Release lists:** **`list_releases`** is the product’s release catalog; **`list_story_map_releases`** is releases **as positioned on that story map** (use before `reorder_story_map_release`).

## Common mistakes

| Symptom | Fix |
|--------|-----|
| Plans from list lines only | `load_source_context` each id. |
| Raw dumps by default | Signals first; raw only if insufficient. |
| Story map / releases errors | Set `clientId` + `productId`. |
| Chat-only “done” | `put_work_item_anchored_context` + version. |
| Tool errors | Human fixes MCP or pastes `tools/list`; no invented args. |
| Write executed without approval | Always call with `confirmed: false` first; show the returned `summary`; only re-call with `confirmed: true` after the user accepts. |
| `pendingOperationId` reused across runs | It is informational — every fresh preview returns a new id. Re-call the **same tool with same args** to execute, not the pending id. |
| Wrong reorder API | **`reorder_story_map_step`** / **`reorder_story_map_activity`** take **`targetPosition`** (integer). **`direction`** is only for **`reorder_story_map_release`**. |
| `get_valid_transitions` with only **`TAL-*`** | Load **`workItemId`** via **`get_work_item`** first; **`get_valid_transitions`** takes **`workItemId`** only. |
| Batch over 200 rows | Split into multiple calls (**200** max per arrays on batch tools). |
| Assuming batch previews validate every row | Preview is a compact summary; ACL and row validation happen on **`confirmed: true`**. Inspect **`failures`** after execute when present. |

## When MCP is down or unclear

Ask the human to restore connectivity or paste **`tools/list`**. Do not invent parameters.
