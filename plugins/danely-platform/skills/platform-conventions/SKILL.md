---
name: platform-conventions
description: >
  Danely platform conventions shared by every danely-* MCP bundle. Use before
  any create/transition/version write — identity model, Latest vs Fixed,
  inspect_actions pre-flight, async command status, idempotency keys, and the
  core-plus-domain bundle connection model.
---

# Danely platform conventions

You are working against **Danely** via the live Azure APIM MCP bundles
(`danely-content`, `danely-decision`, `danely-policy`, `danely-risk`,
`danely-service-management`, `danely-workflow`, `danely-user-admin` →
`https://dnly-apim.azure-api.net/mcp/…`).

These are the **tenant** bundles. The operator perimeter (`subscription`,
`tenant-lifecycle`, `billing-ops`, `support`) is a different route family
— `/mcp/operator/{slug}`, workforce Entra rather than CIAM — and is
deliberately **not** shipped here. Do not reach for a tenant route with an
operator slug: the perimeter filter hides the domain tools and you will read
an empty list as a broken connector.

**Connect to `danely-core` as well as the domain bundle(s) you need.** The
cross-cutting tools — orientation, search, lineage, and the whole async command
protocol — live on `core` alone and are **not** repeated on domain bundles
(ADR-197 Amendment 3). They used to be unioned onto every bundle, which cost one
duplicate tool definition per extra connection: 96 of 112 at seven bundles.

> **A domain-bundle-only connection looks fine until it doesn't.** You can create
> and read perfectly well without `core`. What you cannot do is *finish* an async
> command — `get_command_status` is a core tool — so a `pending` result presents
> as a hang rather than a missing tool. If a command never resolves, check you are
> connected to `core` before diagnosing anything else.

That hostname is the **production APIM perimeter** (not Aspire / not a per-env
switch). Danely is the system of record — prefer MCP tools over guessing from
memory.

Auth is OAuth/CIAM at the connector — if tools 401, reconnect `/mcp` after a
fresh login; do not invent static bearer headers.

## Identity (every aggregate)

- `domainId` — stable identity (use as `targetId` on references)
- `branchId` — line of versions
- `versionId` — one snapshot; **moves on every successful command**

After any command, take the new `versionId` from the result before the next write on that aggregate.

## Targeting writes — Latest vs Fixed

- **Latest (top-level tool args)** — mutate the branch head via `domainId`. Supply
  `branchId` when you know it (safer under multi-branch); when omitted, the server
  may resolve the highest head across branches. Prefer carrying `branchId` once you
  have it from a prior result.
- **Latest (nested `{X}Reference` fields)** — **always** include `branchId` with
  `targetId`. Omitting it fails validation (`requires branchId` / `requires BranchId`).
- **Fixed** — mutate a pinned `versionId`. If the head has moved, use
  `headVersionId` / `suggestedRetry` from the error — do **not** invent a new plan.

Do not use `LatestPublished` as a **write** target; it is a reference/read mode only.

## Pre-flight before transition / version ops

Before any `transition` or versioning write:

1. Call `inspect_actions` on the current version. **`inspect_actions` is a `core`
   tool** — it is not on the domain bundle that owns the aggregate.
2. Fire the trigger **only if** it appears in the returned action descriptors / permitted set.
3. Prefer copying `suggestedInput` when present — but **fill in placeholders**
   (e.g. nested Account `branchId` for `mode: Latest`); suggestedInput is a skeleton,
   not always a complete valid payload.

`inspect_actions` eliminates the type/state-invalid failure class (skipping it can
cost multi-second round trips just to be told "not permitted"). It is **not** a
complete permission oracle: claim-, ownership-, and capability-based denials can
still reject a call that passed pre-flight. Handle those gracefully — do not treat
presence in `actionDescriptors` as a guarantee of success.

## Async commands

Command results are not always terminal:

- statuses include `completed` | `pending` | `failed` | `error` | `declined`
- on `pending`, poll `get_command_status` with the returned `messageStatusId` until a terminal state (or a timeout). **`get_command_status` is a `core` tool** — without a `core` connection you cannot poll at all, and the command will look hung
- on `declined` / `user_declined`, a destructive elicitation was not confirmed — the command never reached the backend; do not treat this as a domain failure
- on timeout / hang, **stop polling** and escalate (optionally search known-issues first) — do not loop forever
- after `completed`, confirm with a read (`get_by_version` / equivalent) when the next step depends on the new head

## Idempotency

- Re-submitting the **same real-world act** (retry after timeout, duplicate send) → **reuse the same `idempotencyKey` GUID**
- A genuinely new act → mint a new key — **`new_idempotency_key` is a `core` tool**

**Known sharp edge (live APIM):** replaying a create with the same key currently
returns an opaque upstream/internal error instead of the original status handle.
The write is usually **not** duplicated, but you cannot recover the ids from the
replay response — fall back to `find_text` / lineage if the first response was lost.
Do not mint a new key thinking the first call failed unless you have verified no
aggregate was created.

## Discovery tax (do this once per session, not every turn)

1. Prefer this skill + `about` / `start_here` over re-deriving conventions. Both are **`core`** tools; `about` still reports the connected domain bundle's aggregates.
2. Use `schema://…` resources when payload shape is unclear.
3. For named graph/query templates (`query_graph`, etc.), use the curated menu — there is no arbitrary Cypher/graph free-for-all.

## Out of scope for this skill

Bundle-specific aggregates and org procedure live in per-bundle skills (Content, Decision, Policy, Risk, Users, Workflow) — load those when the task is domain-specific. Platform mechanics stay here.
