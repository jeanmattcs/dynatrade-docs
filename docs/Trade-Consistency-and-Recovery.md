# Trade Consistency and Recovery

This page summarizes the runtime decision recorded internally as ADR-011: apply real player effects before publishing market pressure, and never automatically retry an ambiguous item delivery.

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

## Pending item delivery

Purchases and sell compensation can create an item-delivery obligation in `pending-deliveries.yml`.

| State | Meaning | Automatic retry |
| --- | --- | --- |
| `PENDING` | No inventory mutation is known to have started. | Yes |
| `IN_PROGRESS` | Persisted immediately before inventory mutation. | Not after restart |
| `PARTIAL` | Some items were delivered; only the exact leftover remains. | Yes, leftover only |
| `DELIVERED` | Terminal outcome; the active record is atomically removed. | No |
| `MANUAL_REVIEW` | Inventory may have changed, but the final result cannot be proven. | No |

After restart, stale `IN_PROGRESS` entries become `MANUAL_REVIEW`. DynaTrade does not redeliver them automatically because the original inventory add may already have succeeded.

## Partial delivery and refunds

If Bukkit accepts only part of a stack, DynaTrade persists the exact leftover. It does not issue an automatic full refund after a confirmed or ambiguous inventory mutation. A full refund after partial delivery would give the delivered portion away for free.

## Sell compensation

When a sell removes items but the economic operation cannot finish, DynaTrade uses pending delivery to return those items. It does not rely on world drops because dropped entities can despawn, be collected by another player, unload with a chunk, or disappear during a crash.

If the compensation cannot be persisted, DynaTrade logs `REFUND_FAILED`. This is an operator incident, not an informational warning.

## Item policy

The current market prices Bukkit material identity. Customized metadata can represent substantially different value, so non-trivial names, lore, enchantments, damage, attributes, persistent data, potion state, custom models, and similar components are rejected.

This prevents custom items from being consumed or normalized as ordinary commodities. Broader item identity is future architecture work.

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
- `MANUAL_REVIEW` currently requires operator investigation.

These are accepted and documented failure modes, not claims of perfect crash atomicity.
