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
Signals for successfully applied trades that were journaled but have not yet been absorbed by a completed cycle.

### Accepted trade journal (`pending-signals.log`)
The append-only durability log for successfully applied trades. It is written before the signal enters the transaction buffer and is used for crash recovery.

### Pending delivery (`pending-deliveries.yml`)
A durable runtime obligation file that can hold item deliveries for purchases or sell compensation, plus pending Vault-credit obligations for completed sell removals.

### Pending runtime retry (`PENDING_RETRY`)
A runtime-state record used when a prepared trade was admitted but not conclusively finished before shutdown or restart recovery.

### `PRE_APPLY`
A retry stage meaning no confirmed inventory or economy mutation has happened yet. The full trade can be retried conservatively later.

### `VAULT_CREDIT_DUE`
A retry stage meaning a sell already removed the items and recovery must only complete the pending Vault credit.

### `JOURNAL_PENDING`
A retry stage meaning player-side effects already happened and recovery must
only finish the durable post-apply journal step.

### `PENDING`
A delivery state that is safe for automatic retry because no inventory mutation is known to have started.

### `IN_PROGRESS`
The state persisted immediately before inventory mutation. A stale `IN_PROGRESS` becomes `MANUAL_REVIEW` after restart.

### `PARTIAL`
A delivery state containing only the exact quantity that Bukkit did not accept.

### `MANUAL_REVIEW`
An ambiguous delivery or Vault-credit mutation that may already have changed player state. It is not retried automatically.

### `REFUND_FAILED`
A high-priority log outcome indicating that sell compensation could not be made durable.

### `APPLY_UNKNOWN`
A runtime state meaning shutdown or crash left the final player-side apply
outcome ambiguous.

### `COMPENSATED`
A runtime state meaning DynaTrade durably recorded compensation for a correlated
ambiguous market signal before removing that signal.

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

### Cycle interval
How often the pricing cycle runs. `20` ticks equals `1` second. Public docs may refer to this conceptually even when the current configuration guide is focused on the safer operator-facing tuning surface.

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
