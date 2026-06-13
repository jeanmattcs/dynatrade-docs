# Glossary

This glossary defines the main terms used throughout DynaTrade's public documentation.

---

## Trade and market activity

### Trade
A player buy or sell action executed through `/buy`, `/sell`, or the market GUI.

### Market signal
An internal record of trade activity for one item and one direction. Signals always carry volume. Player-backed signals can also carry cycle-local participation metadata.

### Transaction buffer
The in-memory structure that accumulates signals between cycles. It also tracks unique player participation per item and side for the current cycle only.

### Pending signals
Signals that were accepted and journaled but have not yet been absorbed by a completed cycle.

### Accepted trade journal (`pending-signals.log`)
The append-only durability log for accepted trades. It is used for crash recovery.

---

## Pricing concepts

### Base price
The configured anchor price of an item from `items.yml`.

### Current price (market price)
The live reference price after cycle calculations are applied.

### Buy price
What a player pays when buying from the market: `current price * (1 + buy-spread)`.

### Sell price
What a player receives when selling to the market: `current price * (1 - sell-spread)`.

### Buy spread
The fractional markup above market price for buys. Default public example: `0.06`.

### Sell spread
The fractional markdown below market price for sells. Default public example: `0.10`.

### Market pressure (`deltaM`)
The net pressure produced by accumulated buy and sell volume for an item in one cycle.

### Player-aware pressure normalization
Base pricing behavior that softens market pressure when too few unique players created the dominant-side volume. If valid participation data exists, DynaTrade applies this automatically. If participation data is missing or invalid, raw pressure is preserved unchanged.

### Target participation players
The number of unique dominant-side players needed for full participation confidence. Default: `4`.

### Minimum participation factor
The lower bound for valid low-participation cycles. Default: `0.25`.

### Active participation reach
An additional refinement that only applies after base participation confidence has already saturated. It compares the dominant-side unique-player count against the currently active economy participant count.

### Reach weight
How strongly active participation reach influences the final normalized factor.

### Sigma
The price sensitivity of an item. Higher sigma means the same pressure moves price more strongly.

### Reference volume (`vref`)
The trade volume scale used when converting raw volume into market pressure.

### Gamma
The mean reversion strength that pulls price back toward baseline.

### Max variation per cycle (`max-var-percent`)
The maximum allowed single-cycle price movement.

### Price floor (`min-price-factor`)
The minimum market price as a multiple of base price.

### Price ceiling (`max-price-factor`)
The maximum market price as a multiple of base price.

### Settle near reference
The final pipeline behavior that snaps a nearly settled result exactly to its baseline.

---

## Cycle and scheduling

### Market cycle
The periodic event that drains the transaction buffer, recalculates prices, persists state, and applies the new runtime prices.

### Cycle interval (`batch-interval-ticks`)
How often the cycle runs. `20` ticks equals `1` second.

### Cycle generation
The incrementing number that advances after every successful cycle.

### Manual cycle
A cycle triggered immediately by an admin using `/dt cycle`.

---

## Idle recovery

### Idle item
An item that has gone several cycles without new trade activity.

### Idle cycle threshold (`idle-cycle-threshold`)
How many quiet cycles must pass before the stronger idle correction can activate.

### Inactive gamma (`inactive-gamma`)
The stronger mean reversion value used during idle recovery.

---

## Economy template

### Template
A predefined market behavior profile such as `STABLE`, `BALANCED`, `VOLATILE`, or `HARDCORE`.

### Overrides
Configuration blocks that customize specific template values without replacing the whole template.

---

## Persistence and recovery

### Market state (`market-state.yml`)
The authoritative persisted market state written after successful cycles.

### Cycle checkpoint (`cycle-checkpoint.yml`)
A prepared cycle result written before the final state commit.

### Checkpoint recovery
The startup process that completes a previously prepared cycle commit after an interrupted shutdown.

### Quarantine
The recovery behavior that skips malformed auxiliary files instead of crashing the runtime.

### Fail-safe
The startup behavior that refuses to silently reset the economy when `market-state.yml` is critically invalid.
