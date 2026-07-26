---
name: platform-conventions
description: >
  Danely platform conventions shared by every danely-* MCP bundle. Use before
  any create/transition/version write — identity model, Latest vs Fixed,
  inspect_actions pre-flight, async command status, idempotency keys.
---

# Danely platform conventions

You are working against **Danely** via the live Azure APIM MCP bundles
(`danely-content`, `danely-decision`, `danely-policy`, `danely-risk`,
`danely-workflow`, `danely-user-admin` → `https://dnly-apim.azure-api.net/mcp/…`).
Danely is the system of record. Prefer MCP tools over guessing from memory.

Auth is OAuth/CIAM at the connector — if tools 401, reconnect `/mcp` after a
fresh login; do not invent static bearer headers.

## Identity (every aggregate)

- `domainId` — stable identity (use as `targetId` on references)
- `branchId` — line of versions (required with `Latest` mode)
- `versionId` — one snapshot; **moves on every successful command**

After any command, take the new `versionId` from the result before the next write on that aggregate.

## Targeting writes — Latest vs Fixed

- **Latest** — mutate the head. Always carry `branchId` with `targetId` (= domainId). Omitting `branchId` often fails with "requires BranchId".
- **Fixed** — mutate a pinned `versionId`. If the head has moved, you get a stale-target error; use `headVersionId` / `suggestedRetry` from the error — do **not** invent a new plan.

Do not use `LatestPublished` as a **write** target; it is a reference/read mode only.

## Pre-flight before transition / version ops

Before any `transition` or versioning write:

1. Call `inspect_actions` on the current version.
2. Fire the trigger **only if** it appears in the returned action descriptors / permitted set.
3. Prefer copying `suggestedInput` when present rather than hand-rolling the payload.

A runtime capability denial after a write is a process failure — the check above should have caught it.

## Async commands

Command results are not always terminal:

- statuses include `completed` | `pending` | `failed` | `error` | `declined`
- on `pending`, poll `get_command_status` with the returned `messageStatusId` until a terminal state (or a timeout)
- on timeout / hang, **stop polling** and escalate (optionally search known-issues first) — do not loop forever
- after `completed`, confirm with a read (`get_by_version` / equivalent) when the next step depends on the new head

## Idempotency

- Re-submitting the **same real-world act** (retry after timeout, duplicate send) → **reuse the same `idempotencyKey` GUID**
- A genuinely new act → mint a new key

## Discovery tax (do this once per session, not every turn)

1. Prefer this skill + the active bundle's `about` / `start_here` over re-deriving conventions.
2. Use `schema://…` resources when payload shape is unclear.
3. For named graph/query templates (`query_graph`, etc.), use the curated menu — there is no arbitrary Cypher/graph free-for-all.

## Out of scope for this skill

Bundle-specific aggregates and org procedure live in per-bundle skills (Content, Decision, Policy, Risk, Users, Workflow) — load those when the task is domain-specific. Platform mechanics stay here.
