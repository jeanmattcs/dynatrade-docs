# Trade Consistency and Recovery

This page summarizes the runtime decision recorded internally as ADR-011: apply real player effects before publishing market pressure, distinguish pre-apply retry work from credit-only retry work, and never automatically retry an ambiguous inventory or Vault mutation.

## Why this decision exists

DynaTrade coordinates Vault, Bukkit inventories, and filesystem persistence. Those systems do not share one atomic transaction. A crash can therefore happen between steps.

The runtime follows this principle:

> When the outcome is ambiguous, never assume success.

This avoids turning an uncertain result into duplicated items or market pressure for a trade that never happened.

## Trade execution order

The current prepared-trade flow is:

```text
validation and admission
  -> bounded apply queue
  -> economy and inventory apply
  -> durable market journal
  -> transaction buffer
  -> pricing cycle
```

Older versions and historical documentation described journal and buffer publication before Bukkit/Vault apply. That ordering could replay price pressure after a trade failed to affect the player.

The current ordering accepts a different failure mode: if the player effect succeeds but the journal later fails, the market may temporarily under-count that trade. DynaTrade prefers missing pressure over pressure created by a nonexistent transaction.

## Runtime retry stages

Prepared-trade runtime recovery now distinguishes two states before it decides
what to replay:

| Stage | Meaning | Automatic path |
| --- | --- | --- |
| `PRE_APPLY` | No confirmed inventory or economy mutation happened yet. | Retry the full trade conservatively when the player is online. |
| `VAULT_CREDIT_DUE` | A sell already removed the items; only the proceeds remain. | Create or resume a pending Vault credit. Never remove items again. |
| `JOURNAL_PENDING` | Player-side effects already happened, but the durable post-apply journal step did not finish before shutdown. | Re-run only the durable journal path and publish the market signal idempotently. |

This distinction is what prevents a restarted sell from charging the player's
inventory twice.

## Pending item delivery and Vault credit

Purchases, sell compensation, and pending sell proceeds can create durable
obligations in `pending-deliveries.yml`.

| State | Meaning | Automatic retry |
| --- | --- | --- |
| `PENDING` | No inventory mutation is known to have started. | Yes |
| `IN_PROGRESS` | Persisted immediately before inventory mutation. | Not after restart |
| `PARTIAL` | Some items were delivered; only the exact leftover remains. | Yes, leftover only |
| `DELIVERED` | Terminal outcome; the active record is atomically removed. | No |
| `MANUAL_REVIEW` | Inventory may have changed, but the final result cannot be proven. | No |

After restart, stale `IN_PROGRESS` entries become `MANUAL_REVIEW`. DynaTrade
does not automatically retry them because the original inventory add or Vault
deposit may already have succeeded.

## Startup recovery order

The current startup path restores pending signals first, promotes unresolved
runtime apply records to `APPLY_UNKNOWN`, compensates correlated ambiguous
signals by persisting durable `COMPENSATED` evidence before removal, and then
replays staged `PENDING_RETRY` work by stage.

That means `JOURNAL_PENDING` recovery happens only after the runtime has a
fresh trade service instance, while `APPLY_UNKNOWN` compensation remains a
startup-only market-signal correction path.

## Partial delivery and refunds

If Bukkit accepts only part of a stack, DynaTrade persists the exact leftover. It does not issue an automatic full refund after a confirmed or ambiguous inventory mutation. A full refund after partial delivery would give the delivered portion away for free.

## Sell compensation and sell-credit recovery

When a sell removes items but the operation later needs a refund, DynaTrade uses
pending item delivery to return those items. It does not rely on world drops
because dropped entities can despawn, be collected by another player, unload
with a chunk, or disappear during a crash.

If the compensation cannot be persisted, DynaTrade logs `REFUND_FAILED`. This is an operator incident, not an informational warning.

When the sell already removed items successfully and only the proceeds are
missing, DynaTrade does not create a new item obligation. It creates a durable
pending Vault credit keyed from the trade `operationId`. If the player is
online on startup, the credit is attempted immediately. If the player is
offline, it remains pending and is completed on join.

## Item policy

The current market prices Bukkit material identity. Customized metadata can represent substantially different value, so non-trivial names, lore, enchantments, damage, attributes, persistent data, potion state, custom models, and similar components are rejected.

This policy is fail-closed. If the runtime sees specialized Bukkit item meta it
does not fully trust, it rejects the trade instead of attempting a partial
interpretation. Broader item identity is future architecture work.

## Reload and shutdown behavior

- shutdown persists queued `PRE_APPLY` retries before discarding apply work
- shutdown also persists abandoned post-apply journal work as `JOURNAL_PENDING`
- reload drains and persists the old runtime first, then stops it completely,
  and only then creates the replacement runtime

Known limitation: reload remains synchronous and can briefly block the main
server thread while it drains in-flight work.

## Concurrency and capacity

- One prepared trade can be in flight per player UUID.
- Different players can trade concurrently.
- Normal delivery obligations are capped at `400`.
- Total delivery obligations are capped at `500`, reserving `100` slots for rollback/refund.
- A normal purchase is rejected before charging when normal capacity is exhausted.

## Operator response

For `MANUAL_REVIEW`:

1. Do not change the state to `PENDING` while the server is running.
2. Preserve `pending-deliveries.yml` and the relevant server logs.
3. Stop the server before any manual file action.
4. Compare player inventory and economy evidence with the delivery ID, item, quantity, reason, and timestamps.
5. Resolve conservatively; current releases do not provide an admin command that can prove the ambiguous outcome.

For `REFUND_FAILED`:

1. Treat it as high priority.
2. Record the operation ID, player UUID, item, quantity, and reason from the log.
3. Fix the storage/capacity problem before attempting manual compensation.
4. Verify the player has not already received the items before compensating.

## Remaining risks

- Vault, Bukkit, and filesystem state are not one atomic transaction.
- Journal failure after successful apply can omit market pressure.
- Pending-delivery YAML persistence is synchronous and can add latency on slow disks.
- At the total cap of `500`, a new compensation cannot be persisted automatically.
- Reload can enter an explicit degraded mode if runtime recreation fails after the old runtime was already stopped.
- `MANUAL_REVIEW` currently requires operator investigation.

These are accepted and documented failure modes, not claims of perfect crash atomicity.
