# Glossary

This glossary defines the terms used throughout DynaTrade's documentation and in-game interfaces. Terms are listed in the order they naturally appear in the system's pipeline, from trade input to price output.

---

## Trade and market activity

### Trade
A player buy or sell action executed through `/buy`, `/sell`, or the market GUI. A trade is validated, charged or paid through Vault, and then recorded as a market signal.

### Market signal
An internal record of trade activity for one item in one direction (buy or sell). Each signal carries a volume — the number of units traded. Signals are collected in the transaction buffer until the next cycle processes them.

### Transaction buffer
An in-memory structure that accumulates market signals between cycles. It is thread-safe and lock-free for recording incoming trades. At cycle time, the buffer is drained atomically — all accumulated signals are extracted at once and replaced with an empty buffer.

### Pending signals
Market signals that have been accepted and journaled but not yet absorbed by a completed cycle. If the server restarts between a trade and the next cycle, pending signals are recovered from the journal and applied at the next cycle.

### Accepted trade journal (`pending-signals.log`)
An append-only log of every accepted trade signal. Written durably after each trade before the signal enters the buffer. Used for crash recovery — if the server stops before a cycle runs, the journal allows DynaTrade to reconstruct what was in the buffer.

---

## Pricing concepts

### Base price
The configured starting price of an item, set in `items.yml`. The base price is the anchor the market reverts toward over time. It does not change dynamically — only `currentPrice` changes.

### Current price (market price)
The live reference price of an item after all cycle calculations are applied. This is the value the market is actually trading around. It can move up or down from the base price based on player activity and mean reversion.

### Buy price
The price a player pays when purchasing an item from the market. Calculated as:
```
buy price = current price × (1 + buy-spread)
```
Always higher than the current price.

### Sell price
The price a player receives when selling an item to the market. Calculated as:
```
sell price = current price × (1 − sell-spread)
```
Always lower than the current price.

### Buy spread
A fractional value added to the market price to determine the buy price. Configured as `pricing.buy-spread` in `config.yml`. A value of `0.06` means players pay 6% above the market price.

### Sell spread
A fractional value subtracted from the market price to determine the sell price. Configured as `pricing.sell-spread`. A value of `0.10` means players receive 10% below the market price.

### Market pressure (deltaM)
The net signal calculated from the accumulated buy and sell volumes for one item in a cycle. High buy volume relative to sell volume produces positive pressure (price rises). High sell volume produces negative pressure (price falls). Balanced activity produces low net pressure.

### Sigma (σ)
The price sensitivity of an item. Controls how strongly market pressure translates into price movement. Higher sigma means the item reacts more aggressively to the same volume of trades. Configurable globally, per category, or per item.

### Reference volume (vref)
The trade volume used as the scale reference when calculating market pressure. A lower vref means a smaller trade volume can move the price significantly. A higher vref requires more trading activity to produce the same price movement. Configurable globally, per category, or per item.

---

## Pricing pipeline steps

### Projected price
The price after applying market pressure and sigma — the raw result before any correction or clamping is applied.

### Mean reversion
A correction applied every cycle that pulls the current price toward the base price. It prevents items from drifting indefinitely away from their configured value. Controlled by the `gamma` parameter.

### Gamma (γ)
The mean reversion strength. Higher gamma pulls prices back to baseline faster. Lower gamma allows prices to stay displaced from their baseline longer.

### Max variation per cycle (`max-var-percent`)
A hard cap on how much the price can change in a single cycle, expressed as a percentage. If the calculated movement exceeds this limit, it is capped. Prevents extreme single-cycle jumps even under very high trade pressure.

### Price floor (`min-price-factor`)
The minimum price an item can reach, calculated as `base-price × min-price-factor`. The market price can never fall below this value regardless of sell pressure. Configurable globally or per item.

### Price ceiling (`max-price-factor`)
The maximum price an item can reach, calculated as `base-price × max-price-factor`. The market price can never rise above this value regardless of buy pressure. Configurable globally or per item.

### Settle near reference
A final pipeline step: if the calculated price after clamping lands very close to the base price, it is snapped exactly to the base price. This prevents prices from hovering just barely away from their anchor indefinitely.

---

## Cycle and scheduling

### Market cycle
The periodic event where DynaTrade drains the transaction buffer, calculates new prices for all affected and recovering items, persists the result, and applies the updated prices to the live runtime. This is the fundamental unit of market progression.

### Cycle interval (`batch-interval-ticks`)
How often the market cycle runs, measured in server ticks. 20 ticks = 1 second. Default is 6000 ticks (5 minutes).

### Cycle generation
An incrementing counter that advances by one after every successful cycle. Used internally to correlate market state, pending signals, and checkpoints. Visible in `/dt status` as "Generation".

### Manual cycle
A cycle triggered immediately by an admin via `/dt cycle`, outside of the normal schedule. Processes all pending signals and updates prices right away.

---

## Idle recovery

### Idle item
An item that has received no new trade signals for several consecutive cycles. An idle item can still be processed for mean reversion if its price remains meaningfully displaced from its base price.

### Idle cycle threshold (`idle-cycle-threshold`)
The number of consecutive quiet cycles an item must have before the accelerated idle recovery rate can activate. Configurable globally, per category, or per item.

### Inactive gamma (`inactive-gamma`)
A stronger mean reversion rate applied to idle items when their price is still significantly displaced from baseline after several quiet cycles. Allows the market to correct old shocks faster without affecting actively traded items.

---

## Economy template

### Template
A preset collection of default values for the pricing engine. Selecting a template in `config.yml` sets sensible defaults for sigma, gamma, vref, max variation, and price factors without requiring manual tuning of every parameter. Available templates: `STABLE`, `BALANCED`, `VOLATILE`, `HARDCORE`.

### Overrides
Configuration blocks in `config.yml` that let you customize specific values within a selected template. Overrides only affect the values you specify — all other template values remain unchanged.

---

## Persistence and recovery

### Market state (`market-state.yml`)
The authoritative persisted record of the current market. Stores the current price of every item, each item's idle cycle count, the current generation number, and a summary of the last completed cycle. Written after every successful cycle and on server shutdown.

### Cycle checkpoint (`cycle-checkpoint.yml`)
A write-ahead record of a cycle that has been fully calculated but not yet completely committed. Written before `market-state.yml` is updated. If the server crashes between writing the checkpoint and updating the market state, DynaTrade recovers the checkpoint on the next startup and completes the commit.

### Checkpoint recovery
The startup procedure where DynaTrade detects a prepared checkpoint and completes the cycle commit that was interrupted. This ensures no cycle result is silently lost after a crash.

### Quarantine
When an auxiliary recovery file (`pending-signals.yml`, `pending-signals.log`, or `cycle-checkpoint.yml`) is malformed or structurally inconsistent, DynaTrade skips it and logs a warning rather than crashing or silently discarding it. The plugin continues from the last clean `market-state.yml`.

### Fail-safe
If `market-state.yml` itself is critically invalid (corrupted, truncated, or structurally broken), DynaTrade refuses to start the market runtime rather than silently resetting the economy to base prices. The admin is expected to intervene.

---

## Market categories

### Category
A market group that items belong to. Each category has a configurable price multiplier and can have its own sigma, vref, inactive-gamma, and idle-cycle-threshold overrides. The six categories are:

| Category | Icon | Default multiplier |
|---|---|---|
| Mining | Iron Pickaxe | ×1.2 |
| Agriculture | Wheat | ×1.0 |
| Miscellaneous | Bundle | ×1.5 |
| Building | Bricks | ×1.0 |
| Magic | Blaze Rod | ×2.0 |
| Alchemy | Glass Bottle | ×1.3 |

The category multiplier is applied to `base-price` to determine the effective starting market price before any dynamic movement occurs.

