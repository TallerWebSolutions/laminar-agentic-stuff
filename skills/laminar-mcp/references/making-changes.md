# Making Changes

## Confirmed two-step contract

Every write tool (except `put_work_item_anchored_context`) requires `confirmed: boolean`:

1. **Preview** — call with `confirmed: false`. Returns `{ confirmed: false, summary, pendingOperationId, expiresAt }`. Echo `summary` to user; wait for explicit approval.
2. **Execute** — call with `confirmed: true`. Returns `{ confirmed: true, ok: true, ... }`.

Never collapse steps. When planning a sequence, run the full preview chain first, then execute in order. If you need new IDs from an earlier step (e.g. `userActivityId` from `create_story_map_activity`), execute that step before previewing the dependent one.

`put_work_item_anchored_context` uses `expectedVersion` (not `confirmed`).

## Handoff (anchored ADR)

`get_work_item_anchored_context` → merge sections (`decisions`, `constraints`, `openQuestions`, `stateItems`, `outOfScope`, `scenarios` + coverage enum, optional `documentTitle`) → `put_work_item_anchored_context` with `expectedVersion` (`0` if new) → on conflict, refetch and retry.

Optional: `update_work_item` via TipTap JSON from `get_work_item` (`descriptionFormat: "json"`).

## Work-item write args

- **`create_work_item`**: `title`, `clientId`, `statusId`, `type`, `serviceClass`; optional `description` (TipTap JSON), `productId`, `releaseId`, `userStepId` (requires `productId` on same product). Resolve ids with list tools — no natural-language resolution.
- **`transition_work_item`**: `workItemQuery`, `toStatusId`. Clears active attributions; reassign after with `assign_work_item`. Call `get_valid_transitions` (`workItemId`) first.
- **`assign_work_item` / `unassign_work_item`**: `workItemQuery`, `teamMemberId` (from `list_team_members`).
- **`update_work_item`**: `workItemId`, optional `title`, `description` (full TipTap JSON), `type`, `serviceClass`, `productId`, `releaseId`.
- **`add_blocker`**: `workItemId`, `blockerTypeId` (from `list_blocker_types`), `reason` (≤ 600 chars); item must be in a **touch** status with no active blocker.
- **`remove_blocker`**: `workItemId`.

## Story map writes

Set product in `set_context`, then `get_story_map` → read activities / steps / releases for current shape.

| Goal | Tool |
|------|------|
| Add activity / step | `create_story_map_activity` (returns `userActivityId`) → `create_story_map_step` (`storyMapId`, `userActivityId`, `name`) |
| Rename / delete | `update_` / `delete_story_map_activity` or `_step` |
| Reorder activities | `reorder_story_map_activity` (`userActivityId`, `targetPosition` 0-based in map) |
| Reorder steps | `reorder_story_map_step` (`userStepId`, `targetPosition` 0-based **within its activity** — read `list_story_map_steps` first) |
| Reorder release strip | `reorder_story_map_release` (`storyMapId`, `releaseId`, `direction`: `up` \| `down`) — `list_story_map_releases` first |
| Move step between activities | `move_step_to_activity` (`userStepId`, `userActivityId`) |
| Place / clear demand on map | `move_work_item_to_step` (`workItemId`, `userStepId` or `null` — does **not** change release) |
| Create demand on map | `create_work_item` with `productId` + optional `userStepId` |

## Batch operations

Use when many similar changes need one preview + one approval.

| Tool | Shared args | Row-level |
|------|-------------|-----------|
| `create_story_map_activities_batch` | `storyMapId` | `activities: [{ name }]` |
| `create_story_map_steps_batch` | `storyMapId` | `steps: [{ name, userActivityId }]` |
| `create_work_items_batch` | `clientId`, `statusId`, `type`, `serviceClass`; optional `productId`, `releaseId` | `items: [{ title, description?, releaseId?, userStepId? }]` — `userStepId` requires shared `productId` |
| `assign_work_items_to_release_batch` | `releaseId` (or `null` to clear) | `workItemQueries: [customId or workItemId]` — release-only, does not move map rows |
| `update_work_items_batch` | — | `items: [{ workItemQuery, ...patch }]` |

**Import order**: `create_story_map_activities_batch` (execute, collect `userActivityId`s) → `create_story_map_steps_batch` (execute, collect `userStepId`s) → `create_work_items_batch`. Inspect `failures` after each execute.

**Session filters**: when `set_context` has pinned `clientId`/`productId`, bulk assign/update rejects rows outside that scope.

## Story-map peers

With product in context: `get_story_map` → `list_story_map_work_items` → filter by `userStepId`, `userActivityName`, or `releaseId` → selectively `get_work_item` / anchored context / `load_source_context` on relevant peers.
