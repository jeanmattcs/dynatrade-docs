# Admin Guide

This page covers day-to-day operations for server administrators running DynaTrade. It explains how to monitor the economy, apply configuration changes safely, respond to problems, and understand the server console output.

---

## Monitoring the economy

### First check: `/dt status`

This is your primary health command. Run it any time you want a quick overview of the economy engine.

```
/dt status
```

A healthy output looks like this:

```
[DynaTrade] -- Status --
[DynaTrade] [SYS] Economy: OK
[DynaTrade] [SYS] Scheduler: active
[DynaTrade] [TIME] Next cycle: 03m 12s

[DynaTrade] [DATA] Generation: 47
[DynaTrade] [DATA] Items: 32
[DynaTrade] [DATA] Pending signals: 4
[DynaTrade] [PRICING] player-aware-pressure: target=6 minParticipation=0.35 activeReachEnabled=true minActiveReach=0.40 reachWeight=0.25
[DynaTrade] [PRICING] last cycle player participation data seen: true

[DynaTrade] [SYS] Checkpoint: none (no pending recovery)
[DynaTrade] [SYS] Storage: healthy (last write OK)
```

| Field | What to look for |
|---|---|
| **Economy** | Should be `OK`. If not, Vault or the economy provider is unavailable. |
| **Scheduler** | Should be `active`. If stopped, the market is not updating. |
| **Next cycle** | Confirms the scheduler is firing normally. |
| **Generation** | Increments by one each cycle. A stale number after a long session may indicate a scheduler problem. |
| **Pending signals** | The number of trades waiting for the next cycle. Normal to see a non-zero value between cycles. |
| **player-aware-pressure** | Shows the current player-aware pricing normalization settings that scale dominant-side pressure. |
| **last cycle player participation data seen** | `true` means the last processed cycle observed participation data. `false` usually means no qualifying participation signal was present in that cycle. |
| **Checkpoint** | Should normally show `none`. A checkpoint present on startup means DynaTrade recovered a prepared cycle — this is expected after a crash. |
| **Storage** | Should be `healthy`. |

If `/dt status` instead prints a degraded-state warning and stops there, the
plugin no longer has an active runtime. Trades are unavailable and a full
server restart is required.

### Checking a specific item: `/dt item <item>`

```
/dt item DIAMOND
```

Use this to answer player questions about a specific item's price, trend, or limits.

```
[DynaTrade] -- Item -- Diamond
[DynaTrade] [PRICE] Current price: $148.20
[DynaTrade] [PRICE] Buy: $157.09 | Sell: $133.38
[DynaTrade] [DATA] Trend: up
[DynaTrade] [DATA] Limits: $45.00 - $1200.00
```

### Optional quote-layer check: `/dt premium status`

Use this only when you are evaluating the optional momentum-driven
quote-adjustment layer rather than the base market price itself.

It shows:

- whether the optional momentum-spread layer is enabled
- the current `tau-*` and `k-*` runtime values
- up to `10` item momentum values with warmup state

If you are not using the optional quote-adjustment layer, you can ignore this
command safely.

---

## Applying configuration changes

### Safe reload flow

1. Edit `config.yml`, `items.yml`, or a language file.
2. Run `/dt reload`.
3. Run `/dt status` — confirm the economy is `OK` and the scheduler is running.
4. Test affected items with `/price <item>` or `/market`.

`/dt reload` preserves the current in-memory market state. Prices are not
reset. The reloaded configuration takes effect for the next cycle and any new
trades.

Operationally, reload now does more than just rebuild config:

- it closes new trade admission first;
- drains in-flight apply and durability work for up to about 3 seconds;
- persists the old runtime before stopping it;
- stops the old runtime completely before creating the replacement runtime;
- keeps the old runtime if the failure happens before the stop boundary;
- enters degraded mode if the failure happens only after the old runtime was already stopped.

If you change `apply.max-per-tick` or `apply.drain-deadline-ms`, treat that as an operational tuning change. Reload applies it immediately, but you should still validate the result during a staged or real load test before leaving it in production.

### When to use reload vs. restart

| Situation | Use |
|---|---|
| Editing `config.yml` or `items.yml` | `/dt reload` |
| Updating the language file | `/dt reload` |
| Adding or removing items | `/dt reload` |
| Updating the DynaTrade jar itself | Full server restart |
| Recovering from a broken or degraded runtime | Full server restart (preferred) or `/dt reset` when the runtime still exists |

### Adding an item mid-session

You can add items to `items.yml` while the server is running:

1. Add the item entry under `items:`.
2. Run `/dt reload`.
3. Test with `/price <NEW_ITEM>`.

The new item will start at its configured base price and begin accumulating market pressure immediately.

### Removing an item mid-session

1. Remove the item entry from `items.yml`.
2. Run `/dt reload`.

Current reload behavior is more conservative than a hard discard. If valid pending signals for the removed item still exist during reload, DynaTrade can preserve the legacy item definition long enough to recover that pending work safely. After that recovery window, the removed item no longer appears in the GUI or responds to commands.

---

## Forcing a market cycle

```
/dt cycle
```

Forces an immediate cycle outside of the normal schedule. All pending buy and sell pressure is processed and prices are updated.

**When to use this:**
- You want to test the effect of recent trades without waiting.
- You are running a controlled test scenario and need deterministic timing.
- You want to give players updated prices after a planned market event.

Forcing a cycle does not reset the scheduler. The next automatic cycle still runs at the normal interval after the forced one.

---

## Resetting the market

```
/dt reset
```

This is a destructive operation that wipes all dynamic price history and resets the market to base prices from `items.yml`.

**Before running reset:**
- Confirm this is what you intend. It cannot be undone.
- Back up `plugins/DynaTrade/market-state.yml` if you want to preserve the current state.

**Reset flow:**
1. Run `/dt reset` — a confirmation token is displayed.
2. Run `/dt reset <token>` within 60 seconds to confirm.
3. DynaTrade clears buffers, deletes recovery files, and rebuilds a clean market state from `items.yml`.
4. The scheduler resumes automatically.

**After reset:**
- Run `/dt status` to confirm the economy is back to `OK`.
- Confirm generation is now `0` or `1`.
- Test with `/price` and `/market`.

---

## Reading the server console

DynaTrade logs to the server console at key events. Learning to read these logs quickly makes monitoring much easier.

### Startup

```
[DynaTrade] [scheduler] started interval=6000t (300s)
[DynaTrade] [runtime] commands registered.
[DynaTrade] [startup] plugin enabled version=0.8.3 language=en template=BALANCED templateOverrides=2 economy=ready restoredItems=32 restoredSignals=3
```

A clean startup shows: the scheduler started, commands registered, and the startup summary reported the restored item/signal counts.

### Cycle summary

```
[DynaTrade] [cycle] generation=42 trigger=TIME items=5 processed=5 buyVolume=320 sellVolume=110 totalVolume=430 status=OK
```

One line per cycle. This is the normal runtime heartbeat. Confirms how many items were priced and the direction of the cycle.

### Trade apply backpressure

The current `0.8.3` line uses bounded main-thread apply draining for durable trades.

```yaml
apply:
  max-per-tick: 8
  drain-deadline-ms: 15
```

Practical guidance:

- keep these defaults unless you have your own benchmark evidence
- raising `max-per-tick` may improve throughput on some machines
- raising it too far can bring back tick pressure during bursts
- lowering the cap or deadline can be gentler on the server, but may increase backlog during heavy trading

### Warnings

```
[DynaTrade] [pending] pending signal payload is missing 'signals'; quarantining payload.
```

A quarantined recovery file means DynaTrade found something it could not read safely and moved it out of the active recovery path instead of crashing. This is an expected defensive behavior after a hard crash or a failed manual edit.

### Errors

```
[DynaTrade] [startup] blocking startup because market-state.yml is invalid.
```

This is a fail-safe condition. DynaTrade has detected that the primary state file is unreadable or structurally broken and has refused to start with potentially corrupted data. See [Troubleshooting](Troubleshooting#market-state-is-invalid) for the recovery procedure.

---

## File reference

| File | Edit it? | Purpose |
|---|---|---|
| `config.yml` | Yes | Global economy configuration |
| `items.yml` | Yes | Item catalog and per-item tuning |
| `languages/messages_en.yml` | Yes | English player-facing text |
| `languages/messages_pt.yml` | Yes | Portuguese player-facing text |
| `market-state.yml` | **No** | Authoritative persisted market state |
| `pending-signals.yml` | **No** | Pending trade signal snapshot |
| `pending-signals.log` | **No** | Accepted trade journal (append-only) |
| `cycle-checkpoint.yml` | **No** | Write-ahead checkpoint for crash recovery |
| `pending-deliveries.yml` | **No** | Pending, partial, manual-review item deliveries, plus pending Vault-credit obligations |

### Pending delivery incidents

`MANUAL_REVIEW` means the inventory may already have changed and DynaTrade cannot prove the final outcome. It is intentionally excluded from automatic retry.

Do not change `MANUAL_REVIEW` to `PENDING` on a running server. Stop the server, back up the file and logs, and verify player inventory/economy evidence before manual resolution.

`REFUND_FAILED` means a sell compensation could not be persisted. Record the operation ID, player UUID, item, quantity, and reason; fix storage or capacity first; then verify that compensation was not already delivered before taking manual action.

If the console reports degraded mode after `/dt reload`, do not keep issuing
trade commands. The old runtime is already gone and a restart is required to
rebuild the runtime safely.

See [Trade Consistency and Recovery](Trade-Consistency-and-Recovery.md#operator-response) for the complete procedure.

---

## Permission delegation

DynaTrade uses two admin permission nodes intentionally. This lets you give staff diagnostic access without the ability to wipe the economy.

| Permission | What it enables |
|---|---|
| `dynatrade.admin` | Status, item inspection, premium status, reload, manual cycle |
| `dynatrade.admin.reset` | Full market reset (destructive — give to owners only) |

---

## Recommended admin routine

| Frequency | Action |
|---|---|
| After every configuration change | `/dt reload`, `/dt status`, spot-check one item with `/dt item` |
| After a crash or unexpected restart | Check startup logs, run `/dt status`, confirm generation advanced correctly |
| When players report wrong prices | `/dt item <item>`, compare to `items.yml` base price and limits |
| When the economy feels stuck | `/dt status` to check scheduler, `/dt cycle` to force a cycle if needed |
| Before a planned economy wipe | Back up `market-state.yml`, then run `/dt reset` |


