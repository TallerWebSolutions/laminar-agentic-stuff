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

**5. Draft** — apply title + body templates (see [Demand conventions](#demand-conventions))

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

**Starting work on a demand** — after reading it, always ask:
1. *"Should I assign you to this demand?"* → `list_team_members` to find the user's `teamMemberId` → `assign_work_item` (`confirmed: false` preview first)
2. *"Should I move it to the active doing status?"* → `get_valid_transitions` → `transition_work_item` to the appropriate `touch`-type status

## Close demand (after ship / production push)

Follow in order.

**1. Sync anchored context**
`get_work_item_anchored_context` → if everything was actually done, mark every `stateItem` done, every scenario `covered`, clear `openQuestions` → `put_work_item_anchored_context` with `expectedVersion`.
On `ANCHORED_CONTEXT_VERSION_CONFLICT`: refetch and retry with the new version.

**2. Sync demand description** *(if AC changed during implementation)*
`get_work_item` (`descriptionFormat: "json"`) → merge new scenarios into the TipTap JSON → `update_work_item`. The demand description is what QA reads; the anchored context is what the next dev reads — both must reflect the full shipped scope.

**3. Walk to final Done** *(ask the user first, if should move to final done)*
`get_valid_transitions` → `transition_work_item` → repeat until a status with `type: final` named "Done" is reachable. Workflows often gate the final status with intermediate queue/touch steps — advance one hop at a time.

## Hours pressure report

Use when asked to compute per-client hours pressure for a month — same calc as the Status Report "Horas" tab.

**Consumed hours** (mirrors UI Status Report "Horas"): `consumed = attribution.hours * (1 + overhead/100) + activity.hours + manual.hours`. Default `overhead = 20`.

**Pressure per client** as decimal ratio, weighted by share of team capacity:

```
pressure = ((reserved * percentMonthElapsed - consumed) / reserved) * (reserved / totalCapacityHours)
```

`totalCapacityHours` is the team's total available hours for the month (required param). `percentMonthElapsed = elapsedBusinessDays / totalBusinessDays` (Excel NETWORKDAYS semantics: Mon–Fri UTC minus any `holidays`). Positive = slack, negative = deficit. Example: reserved=100, consumed=35, percentMonth=1, totalCapacity=500 → `(1 - 0.35) * 0.2 = 0.13` (13% slack against the whole team). `totals.pressure` is the sum across clients, also expressed as ratio of team capacity.

**Flow**
1. `list_clients` → resolve client IDs.
2. (Per-product filter) `list_products` (set client context first) → identify product IDs (e.g. "Sites taller" for Taller).
3. Ask the user for each client's `reservedHours` for the target month, the team's `totalCapacityHours`, and any `holidays` (optional ISO `YYYY-MM-DD` array).
4. Call `get_clients_pressure` with `period: "YYYY-MM"`, `totalCapacityHours`, `holidays?`, and `inputs: [{clientId, clientName?, productIds?, reservedHours}, ...]`.
5. For a single-client consumed-only number: `get_consumed_hours` (skip step 3).

**Notes**
- Both tools enforce client view ACL via PAT user.
- `percentMonthElapsed` counts only business days (Mon-Fri UTC) minus `holidays`. For a fully past period it is `1`.
- Override overhead per call with `overheadPercentual`.
- Response also returns `totalBusinessDays` and `elapsedBusinessDays` for transparency.

## Demand conventions

**Language**: Match the conversation's language by default. At `confirmed: false` preview, ask: *"Translate to [language] before creating?"*

**Title + body**: Apply templates in [references/demand-templates.md](references/demand-templates.md) — title slot pattern (Quem/Onde/Quando/O que/Para que), body sections in order (Problema required, Apoio optional, Critérios de aceite required), Gherkin AC required with keywords matching the demand's language (pt-BR demand → `Funcionalidade`/`Cenário`/`Dado`/`Quando`/`Então`; English demand → `Feature`/`Scenario`/`Given`/`When`/`Then`).

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
| Handoff not persisted | `put_work_item_anchored_context` + `expectedVersion` — chat summary alone is not persisted |
| Status left open after ship | Walk `get_valid_transitions` → `transition_work_item` to final Done — don't stop at the first "Done" queue step |
| Final "Done" not in `get_valid_transitions` | Workflow gates it behind intermediate steps — advance one hop at a time until `type: final` is reachable |
| New scenarios only in anchored context | `update_work_item` (descriptionFormat: json) too — the demand description is what QA reads |
| Demand not assigned when work starts | `list_team_members` → `assign_work_item` — always ask at the start of a work session |
| `ANCHORED_CONTEXT_VERSION_CONFLICT` on put | Refetch with `get_work_item_anchored_context`, merge locally, retry with the new `expectedVersion` |
| Wrong reorder API | `reorder_story_map_step` / `_activity` take `targetPosition` (integer); `direction` is only for `reorder_story_map_release` |
| Title missing Quem/Onde/Quando/O que slot | Refer to [references/demand-templates.md](references/demand-templates.md) — all slots except "Para que" are mandatory |
| Gherkin keyword language mismatches demand body (e.g. pt-BR demand with `Given`/`When`/`Then`) | Match demand language — pt-BR demand uses `Funcionalidade`/`Contexto`/`Cenário`/`Dado`/`Quando`/`Então`/`E`/`Mas`; English demand keeps `Feature`/`Background`/`Scenario`/`Given`/`When`/`Then`/`And`/`But` |
