# How the Market Works

DynaTrade is not a shop with fixed prices. It is an economic engine where prices are driven by real player activity and updated in scheduled batches.

This page explains the full model: how trades become pressure, how pressure becomes price movement, and how the market state is kept safe across restarts and crashes.

---

## The cycle model

Every price change in DynaTrade passes through a cycle. A cycle is a scheduled event that:

1. Collects all buy and sell activity since the last cycle
2. Calculates price movement for each affected item
3. Applies mean reversion to items that have drifted from their baseline
4. Saves the updated market state to disk
5. Applies the new prices to the live runtime

Prices do not change after every transaction. They change when the next cycle runs.

The cycle interval is controlled by `batch-interval-ticks` in `config.yml`. The default is every 5 minutes (6000 ticks). You can force an immediate cycle with `/dt cycle`.

---

## The three-price model

Each tradable item has three visible values:

| Value | What it means |
|---|---|
| **Market price** | The internal reference price — the economic signal |
| **Buy price** | What players pay when buying: `market price × (1 + buy-spread)` |
| **Sell price** | What players receive when selling: `market price × (1 − sell-spread)` |

Example with default spreads (`buy-spread: 0.06`, `sell-spread: 0.10`):

```
Market price: 100
Buy price:    106   (player pays this)
Sell price:    90   (player receives this)
```

The spread creates natural friction. Players cannot instantly buy and sell the same item for profit. It also gives the market room to breathe without forcing large price gaps.

---

## How trades create market pressure

When a player buys or sells an item, the trade is recorded as a **market signal** — a record of buy or sell volume for that item.

These signals are held in a buffer until the next cycle runs. Nothing about the price changes yet.

At cycle time, DynaTrade looks at the accumulated buy and sell volume for each item and calculates a pressure value (`deltaM`). This is the key input to the pricing engine.

- High buy volume relative to sell volume → upward pressure → price rises
- High sell volume relative to buy volume → downward pressure → price falls
- Balanced activity → low net pressure → price moves little

---

## The pricing pipeline

For each item with pending signals, the cycle runs this pipeline:

```
1. Calculate market pressure (deltaM) from buy/sell volume vs. reference volume (vref)
2. Project the new price based on sigma (volatility) and deltaM
3. Apply mean reversion — pull the price back toward its configured baseline
4. Apply max variation cap — limit how far the price can move in a single cycle
5. Clamp the result to the item's configured floor and ceiling (min/max price factor)
6. Settle near the reference if the result lands very close to the baseline
```

Items without new trade activity in a given cycle can still be processed for mean reversion if their price remains displaced from baseline.

---

## Key tuning parameters

| Parameter | What it controls |
|---|---|
| `sigma` | How strongly the item reacts to buy/sell pressure. Higher = more sensitive. |
| `vref` | Reference volume. Lower = smaller trades can move the price. |
| `gamma` | Mean reversion strength. Higher = prices return to baseline faster. |
| `max-var-percent` | Maximum price movement allowed per cycle (e.g. `0.15` = 15% cap). |
| `min-price-factor` | Price floor as a multiple of base price. |
| `max-price-factor` | Price ceiling as a multiple of base price. |

These can be set globally, per category, or per item. See [Configuration](Configuration.md) for full details.

---

## Mean reversion and idle recovery

DynaTrade includes two mechanisms for pulling prices back toward their baseline:

**Regular mean reversion** runs every cycle. It applies a small pull toward the base price regardless of trade activity. Controlled by `gamma`.

**Idle recovery** activates when an item has been inactive for several consecutive cycles and its price is still meaningfully displaced from its base. It applies a stronger snapback force to prevent prices from staying stuck at an extreme value after a market shock.

Idle recovery thresholds are configured with:
- `idle-cycle-threshold` — how many quiet cycles before idle recovery can accelerate
- `idle-deviation-percent-threshold` — minimum relative displacement to trigger acceleration
- `idle-deviation-absolute-threshold` — minimum absolute displacement to trigger acceleration
- `inactive-gamma` — the stronger reversion rate used during idle recovery

---

## Market state persistence

After every cycle, DynaTrade writes the updated market state to disk. The key files are:

| File | Purpose |
|---|---|
| `market-state.yml` | The authoritative record of current prices, idle cycle counts, and cycle generation |
| `pending-signals.yml` | Snapshot of signals that have been accepted but not yet processed by a cycle |
| `pending-signals.log` | Append-only journal of accepted trade signals — used for recovery |
| `cycle-checkpoint.yml` | Write-ahead record of a cycle result that has been calculated but not yet fully committed |

**Do not edit these files manually.** They are managed exclusively by DynaTrade.

---

## Recovery and crash safety

DynaTrade is designed to recover cleanly from crashes, hard reboots, and mid-cycle failures.

### On startup, DynaTrade:

1. Loads `market-state.yml` — the last confirmed clean state
2. Checks for a prepared `cycle-checkpoint.yml` — if a cycle was completed but not fully committed before shutdown, DynaTrade recovers it and completes the commit
3. Recovers any accepted trade signals from `pending-signals.log` that were not yet absorbed by a cycle

### Fail-safe behavior

If `market-state.yml` is critically invalid (corrupted, structurally broken, or unreadable), DynaTrade **does not silently reset the economy**. It logs the problem clearly and prevents the runtime from starting with bad state.

This protects servers from silent price resets after a failed manual edit or corrupted save.

### Quarantine behavior

Auxiliary recovery files (`pending-signals.yml`, `pending-signals.log`, `cycle-checkpoint.yml`) that are malformed or structurally inconsistent are quarantined rather than blocking startup. DynaTrade logs what was skipped and continues from the last confirmed clean state.

---

## The full cycle flow

```
Player buys or sells
        ↓
Trade is validated and executed (economy + inventory)
        ↓
Signal is written to the accepted trade journal (durable)
        ↓
Signal is recorded in the in-memory transaction buffer
        ↓
[next cycle trigger]
        ↓
Buffer is drained and snapshotted
        ↓
Pricing pipeline runs per item
        ↓
Cycle checkpoint is written to disk
        ↓
market-state.yml is updated with new prices
        ↓
In-memory prices are updated (players now see new prices)
        ↓
Checkpoint and auxiliary signals are cleaned up
```

The checkpoint is written before the final state save. This ensures that even if the server crashes between writing the new state and cleaning up, the checkpoint allows DynaTrade to complete the cycle cleanly on the next startup.

---

## Summary

- Prices update in cycles, not per transaction
- Cycles aggregate all pending buy/sell pressure and apply a bounded, controlled price change
- Mean reversion keeps prices from drifting indefinitely
- Every cycle result is saved durably before being applied
- DynaTrade recovers safely from crashes without silently resetting the economy

