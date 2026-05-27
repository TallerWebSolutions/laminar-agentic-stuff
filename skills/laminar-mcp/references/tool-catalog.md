# Tool Catalog

> Server `tools/list` (or equivalent) wins on names and args over this file. Assume **only** MCP access — not a checkout, workspace, or app shell.

## MCP tools

| Area | Tools |
|------|--------|
| **Session** | `get_current_context`, `set_context` (`clientId`?, `productId`?), `clear_context` |
| **Source context** | `list_source_contexts` (`limit`?), `load_source_context` (`sourceContextIds`, `includeRawText`?) |
| **Portfolio / org lists** | `list_clients`, `list_products` (`clientId`? or session client), `list_statuses`, `list_team_members`, `list_blocker_types`, `list_releases` (session `productId` or `productId` arg) |
| **Work items (read)** | `list_work_items` (`completionFilter`?: `open` default \| `all`), `get_work_item` (`query`, `descriptionFormat`?: `plain` to read \| `json`/`both` only if editing description), `get_valid_transitions` (`workItemId`) |
| **Story map (read)** | `get_story_map` (`productId`?), `list_story_map_activities` (`storyMapId`), `list_story_map_steps` (`storyMapId`), `list_story_map_releases` (`storyMapId`), `list_story_map_work_items` (`productId`?) |
| **Work items (write)** | `create_work_item`, `transition_work_item`, `assign_work_item`, `unassign_work_item`, `update_work_item`, `add_blocker`, `remove_blocker` |
| **Story map (write)** | `create_story_map_activity`, `update_story_map_activity`, `delete_story_map_activity`, `create_story_map_step`, `update_story_map_step`, `delete_story_map_step`, `move_step_to_activity`, `move_work_item_to_step`, `reorder_story_map_step`, `reorder_story_map_activity`, `reorder_story_map_release` |
| **Batch writes** | `create_story_map_activities_batch`, `create_story_map_steps_batch`, `create_work_items_batch`, `assign_work_items_to_release_batch`, `update_work_items_batch` — max **200** rows |
| **Anchored ADR** | `get_work_item_anchored_context` (`query`), `put_work_item_anchored_context` (`workItemQuery`, structured sections, `expectedVersion` — not `confirmed`) |
| **Hours / pressure** | `get_consumed_hours` (`clientId`, `productIds`?, `startDate`, `endDate`, `overheadPercentual`? default `20`) → `consumed` in hours; `get_clients_pressure` (`period: "YYYY-MM"`, `totalCapacityHours`, `overheadPercentual`? default `20`, `holidays`? array of `"YYYY-MM-DD"`, `inputs: [{clientId, clientName?, productIds?, reservedHours}]`) → per-client `pressure = ((reserved * percentMonth - consumed) / reserved) * (reserved / totalCapacityHours)` as decimal ratio of team capacity. `percentMonth = elapsedBusinessDays / totalBusinessDays` (NETWORKDAYS) |

## ID and query conventions

- **`query` / `workItemQuery`**: the work item's `customId` (org-specific format, e.g. `PROJ-123`) or its internal `workItemId`; used by `get_work_item`, `get_work_item_anchored_context`, `put_work_item_anchored_context`, `transition_work_item`, `assign_work_item`, `unassign_work_item`.
- **`workItemId`**: the internal `workItemId` value from `get_work_item` / `list_work_items` (not the `customId`); required by `update_work_item`, `add_blocker`, `remove_blocker`, `move_work_item_to_step`, `get_valid_transitions`.
- **Creates** return new ids (e.g. `create_work_item` → `workItemId`, `customId`; story-map creates → `userActivityId` / `userStepId`; `add_blocker` → `blockerId`).

## Supporting reads

Before writes: `get_valid_transitions` (`workItemId`) before `transition_work_item`; `list_blocker_types` before `add_blocker`; `list_team_members` before assign/unassign. `list_work_items` defaults to `open`; pass `completionFilter: "all"` when you need done items.

**Release lists**: `list_releases` is the product's release catalog; `list_story_map_releases` is releases as positioned on the map (use before `reorder_story_map_release`).

## For the collaborator

- Attach `@laminar-mcp` when starting demand work.
- Give demand id (`TAL-*` or `workItemId`) and client/product scope when it must not be guessed; say if you want anchored handoff at session end.
- If MCP fails: confirm MCP is enabled + auth; retry or paste `tools/list` for the agent.

## When MCP is down

Ask the human to restore connectivity or paste `tools/list`. Do not invent parameters.
