# How the Market Works

DynaTrade is not a fixed-price shop. It is an economic engine where prices respond to real player activity and update in scheduled cycles.

This page explains how trades become pressure, how pressure becomes price movement, and how the runtime stays safe across restarts and crashes.

---

## The cycle model

Every price change in DynaTrade passes through a cycle. A cycle:

1. Collects all buy and sell activity since the last cycle
2. Calculates price movement for each affected item
3. Applies mean reversion to items that drifted from baseline
4. Saves the updated market state to disk
5. Applies the new prices to the live runtime

Prices do not change after every trade. They change when the next cycle runs.

The cycle interval is controlled by `batch-interval-ticks` in `config.yml`. The default is every 5 minutes (`6000` ticks). You can force an immediate cycle with `/dt cycle`.

---

## The three-price model

Each tradable item has three visible values:

| Value | What it means |
| --- | --- |
| **Market price** | The internal reference price used by the economic model |
| **Buy price** | What players pay: `market price * (1 + buy-spread)` |
| **Sell price** | What players receive: `market price * (1 - sell-spread)` |

Example with default spreads (`buy-spread: 0.06`, `sell-spread: 0.10`):

```text
Market price: 100
Buy price:    106
Sell price:    90
```

The spread creates friction so players cannot instantly buy and sell the same item for profit.

---

## How trades create market pressure

When a player buys or sells an item, the trade becomes a market signal for that item and direction.

Signals are held in the transaction buffer until the next cycle runs. At cycle time, DynaTrade compares accumulated buy and sell volume and calculates market pressure (`deltaM`).

- High buy volume relative to sell volume creates upward pressure
- High sell volume relative to buy volume creates downward pressure
- Balanced activity creates low net pressure

DynaTrade also considers how many unique players contributed to the dominant side of that pressure in the current cycle.

This is base pricing behavior in the current `0.8.2` line:

- if valid participation data exists, pressure is normalized automatically
- if participation data is missing or invalid, raw pressure is preserved exactly

This lets DynaTrade distinguish one-player bulk volume from broader server participation with the same total volume.

The current line also includes active participation reach. Once the cycle already has full participation confidence, DynaTrade can soften that confidence when the same unique-player count represents only a small share of the currently active economy participants.

Separately, the current runtime can also compute a momentum signal from recent completed cycles. That signal does not change the core market price by itself. It is used only by the optional quote-adjustment layer when that layer is enabled.

---

## The pricing pipeline

For each item with pending signals, the cycle runs this pipeline:

```text
1. Calculate market pressure (deltaM) from buy/sell volume vs. reference volume (vref)
2. Normalize pressure by current-cycle player participation, when participation data exists
3. Optionally refine full-confidence participation using active participation reach
4. Project the new price based on sigma and adjusted deltaM
5. Apply mean reversion toward the configured baseline
6. Apply the max variation cap for that cycle
7. Clamp the result to the configured floor and ceiling
8. Settle exactly on the baseline if the result lands very close to it
```

Items without new trade activity in a cycle can still be processed for mean reversion if their price remains displaced from baseline.

---

## Key tuning parameters

| Parameter | What it controls |
| --- | --- |
| `sigma` | How strongly the item reacts to buy/sell pressure |
| `vref` | Reference volume for pressure scaling |
| `player-aware-pressure.target-participation-players` | Unique players on the dominant side needed for full participation confidence |
| `player-aware-pressure.min-participation-factor` | Minimum factor used when participation exists but is still below target |
| `player-aware-pressure.active-participation-reach.enabled` | Turns the active-reach refinement on or off without disabling base player-aware normalization |
| `player-aware-pressure.active-participation-reach.min-active-reach-factor` | Lower bound for the active-reach factor |
| `player-aware-pressure.active-participation-reach.active-participant-floor` | Minimum active-economy participant count considered by the reach layer |
| `player-aware-pressure.active-participation-reach.active-participant-cap` | Maximum active-economy participant count considered by the reach layer |
| `player-aware-pressure.active-participation-reach.reach-weight` | How strongly active reach influences the final factor |
| `gamma` | Mean reversion strength |
| `max-var-percent` | Maximum allowed price movement in one cycle |
| `min-price-factor` | Price floor relative to base price |
| `max-price-factor` | Price ceiling relative to base price |

These can be set globally, per category, or per item. See [Configuration](Configuration.md) for details.

---

## Mean reversion and idle recovery

DynaTrade uses two mechanisms to pull prices back toward baseline:

`Mean reversion`
Applies every cycle and gently pulls the price toward its base value.

`Idle recovery`
Applies a stronger correction when an item has been inactive for several cycles and is still meaningfully displaced from baseline.

Idle recovery tuning:

- `idle-cycle-threshold`
- `idle-deviation-percent-threshold`
- `idle-deviation-absolute-threshold`
- `inactive-gamma`

---

## Market state persistence

After every cycle, DynaTrade writes state to disk:

| File | Purpose |
| --- | --- |
| `market-state.yml` | Authoritative current market state |
| `pending-signals.yml` | Snapshot of accepted but not yet processed signals |
| `pending-signals.log` | Append-only accepted trade journal for recovery |
| `cycle-checkpoint.yml` | Write-ahead record of a prepared cycle result |

Do not edit these files manually.

---

## Recovery and crash safety

On startup, DynaTrade:

1. Loads the last confirmed `market-state.yml`
2. Checks for a prepared `cycle-checkpoint.yml`
3. Restores accepted trade signals from `pending-signals.log` when needed

If `market-state.yml` is critically invalid, DynaTrade does not silently reset the economy. It blocks startup of the market runtime instead.

Malformed auxiliary recovery files are quarantined instead of crashing the runtime.

---

## Full flow

```text
Player buys or sells
  -> trade is validated and executed
  -> accepted trade is journaled durably
  -> signal enters the in-memory transaction buffer
  -> durable runtime apply is queued
  -> Bukkit-side apply completes
  -> next cycle drains the buffer
  -> pricing pipeline runs per item
  -> cycle checkpoint is written
  -> market-state.yml is updated
  -> in-memory prices are replaced
  -> checkpoint and auxiliary pending state are cleaned up
```

---

## Summary

- Prices update in cycles, not per transaction
- Trade volume becomes market pressure
- Player participation can soften that pressure automatically
- Active participation reach can refine fully-saturated participation
- Recovery preserves accepted trades across restarts and crashes
