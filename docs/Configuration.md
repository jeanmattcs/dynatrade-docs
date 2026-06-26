# Configuration

DynaTrade is configured through two main files plus optional language files.

| File | Purpose |
| --- | --- |
| `plugins/DynaTrade/config.yml` | Global market behavior, template selection, spreads, sounds, apply controls, and operational logs |
| `plugins/DynaTrade/items.yml` | The tradable item catalog and per-item tuning |
| `plugins/DynaTrade/items_pt.yml` | Optional Portuguese item-name overrides for server-side item presentation |
| `plugins/DynaTrade/languages/messages_en.yml` | English player-facing and GUI text |
| `plugins/DynaTrade/languages/messages_pt.yml` | Portuguese player-facing and GUI text |

After editing any of these files, run `/dt reload`.

If `/dt status` later reports degraded mode after a failed reload, restart the
server before resuming trading. That means the old runtime was already stopped
and the replacement runtime could not be created.

Do not manually edit `market-state.yml`, `pending-signals.yml`, `pending-signals.log`, `cycle-checkpoint.yml`, or `pending-deliveries.yml`. Those files contain live runtime state managed by DynaTrade.

---

## `config.yml`

### Language

```yaml
language: en
```

Bundled values:

- `en`
- `pt`

---

### Economy template

```yaml
economy:
  template: BALANCED
```

Templates define the default market personality.

| Template | Behavior |
| --- | --- |
| `STABLE` | Slow and predictable price movement |
| `BALANCED` | Recommended default for most servers |
| `VOLATILE` | More visible reactions to player pressure |
| `HARDCORE` | Aggressive price movement and sharper swings |

If you are unsure, start with `BALANCED`.

---

### Economy overrides

Use overrides when you want to customize the selected template instead of replacing it entirely.

```yaml
economy:
  overrides:
    market: {}
    categories: {}
    category-behavior: {}
    pricing: {}
```

Useful category-level tuning:

```yaml
economy:
  overrides:
    category-behavior:
      AGRICULTURE:
        sigma: 0.22
        vref: 240
        inactive-gamma: 0.28
        idle-cycle-threshold: 4
```

Supported categories:

- `MINING`
- `AGRICULTURE`
- `MISCELLANEOUS`
- `BUILDING`
- `MAGIC`
- `ALCHEMY`

---

### Buy and sell spread

```yaml
pricing:
  buy-spread: 0.06
  sell-spread: 0.10
```

Example at market price `100`:

- buy price = `106`
- sell price = `90`

Higher spread increases friction. Lower spread makes trading closer to the market reference price.

---

### Idle recovery

```yaml
pricing:
  idle-cycle-threshold: 7
  idle-deviation-percent-threshold: 0.15
  idle-deviation-absolute-threshold: 0.20
  inactive-gamma: 0.20
```

Idle recovery helps stale items move back toward their baseline when they have been quiet for several cycles.

---

### Player-aware pressure normalization

```yaml
pricing:
  player-aware-pressure:
    target-participation-players: 4
    min-participation-factor: 0.25
    active-participation-reach:
      enabled: true
      min-active-reach-factor: 0.50
      active-participant-floor: 4
      active-participant-cap: 30
      reach-weight: 0.25
```

Player-aware pressure normalization is part of DynaTrade's base pricing model. It does not need to be enabled.

The top-level values tune the base participation normalization:

| Key | Meaning |
| --- | --- |
| `target-participation-players` | Unique players on the dominant pressure side needed for full pressure in one cycle |
| `min-participation-factor` | Minimum pressure factor when participation data exists but is below the target |

Default behavior:

| Unique players | Pressure factor |
| ---: | ---: |
| 1 | `0.25` |
| 2 | `0.50` |
| 3 | `0.75` |
| 4+ | `1.00` |

When participation data is not available, DynaTrade uses the original raw pressure unchanged. This covers recovery replay, older aggregate signals, and internal signals without player identity.

The nested `active-participation-reach` block tunes the additional reach refinement:

| Key | Meaning |
| --- | --- |
| `enabled` | Enables or disables only the active-reach refinement layer |
| `min-active-reach-factor` | Lower bound for the active-reach factor |
| `active-participant-floor` | Minimum active-economy participant count considered by the reach layer |
| `active-participant-cap` | Maximum active-economy participant count considered by the reach layer |
| `reach-weight` | How strongly active reach influences the final factor |

Important behavior:

- `enabled: false` disables only the active-reach refinement layer
- base player-aware normalization remains part of pricing behavior
- the reach layer only refines cycles that already reached full participation confidence
- invalid or unavailable active-participant data falls back automatically to the base participation result

With the current defaults, `4/4` active participants stays at full confidence while `4/30` is softened to a final factor of `0.875`.

---
### Minimal effectiveVref calibration

The current `0.8.4` technical preview also includes minimal effectiveVref calibration.

```yaml
pricing:
  calibration:
    enabled: true
    smoothing-alpha: 0.25
    max-effective-vref-multiplier: 8
```

What this does today:

- keeps common high-volume items from overreacting to normal volume
- updates calibration only during pricing cycles
- falls back to configured `vref` when calibration is disabled, missing, or empty after restart

What it does not do today:

- no `riskScore`
- no volatility profiles
- no thin-market detection
- no `adaptiveSpreadBonus`
- no admin calibration diagnostics

This is intentionally a bounded first slice. Remaining calibration scope (risk scoring, volatility profiles, thin-market detection, adaptive spread behavior, and admin diagnostics) is future work.

---
### Operational logs

```yaml
pricing:
  batch-tick-log-enabled: false
  batch-empty-log-enabled: false
  batch-drain-summary-log-enabled: true
  batch-pricing-summary-log-enabled: true
```

Recommended production posture:

- keep tick and empty logs off
- keep summary logs on
- enable extra logs only when diagnosing behavior

---

### Apply backpressure

The `0.8.4` line includes bounded main-thread apply draining for durable trades.

```yaml
apply:
  max-per-tick: 8
  drain-deadline-ms: 15
```

What these do:

| Key | Meaning |
| --- | --- |
| `max-per-tick` | Maximum durable trades applied on the Bukkit thread in one tick |
| `drain-deadline-ms` | Maximum time budget for one apply drain pass |

Validated production defaults:

- `max-per-tick: 8`
- `drain-deadline-ms: 15`

Change these only if you are actively benchmarking your own environment.

---

### Trade history retention

```yaml
history:
  inactive-player-ttl-days: 7
```

What this does:

| Key | Meaning |
| --- | --- |
| `inactive-player-ttl-days` | Removes cached trade-history state for players who have not been read or indexed for that many days |

Notes:

- use `0` to disable this TTL-based eviction
- this does not change live prices or pending trade state
- it keeps per-player history retention bounded over time

---

### Trade admission control

The current runtime also includes overload protection for accepted-but-not-yet-applied trades.

```yaml
trade:
  admission:
    enabled: true
    max-pending-applies-global: 5000
    max-pending-applies-per-player: 50
    reject-when-full: true
    backlog-guard:
      enabled: true
      reject-when-backlog-above: 4500
      accept-when-backlog-below: 2500
```

What these do:

| Key | Meaning |
| --- | --- |
| `enabled` | Turns admission control on or off |
| `max-pending-applies-global` | Hard cap for pending applies across the whole server |
| `max-pending-applies-per-player` | Per-player cap for pending applies |
| `reject-when-full` | Rejects new trades instead of allowing unbounded queue growth |
| `backlog-guard.*` | Hysteresis guard that rejects while backlog is high and reopens when it falls |

Recommended posture:

- keep these defaults unless you are running your own load validation
- treat this as server protection, not economy tuning

---

### Momentum signal

The current runtime can compute a recent-cycle momentum signal per item.

```yaml
momentum:
  enabled: true
  window_size: 3
  smoothing_alpha: 0.7
  sensitivity: 0.20
  max_change_per_cycle: 0.30
```

Important:

- this signal does not change the core market price directly
- it exists to summarize recent cycle direction in a bounded way
- it is consumed by the optional quote-adjustment layer when that layer is enabled

---

### Optional quote-adjustment layer

The current config still uses the internal compatibility key `premium`, but this is best understood as an optional quote-adjustment layer.

```yaml
premium:
  momentum-spread:
    enabled: false
    tau-base: 0.05
    tau-min: 0.01
    tau-max: 0.20
    k-sell: 0.10
    k-buy: 0.10
```

What this means:

- disabled by default
- can widen or tighten buy and sell spreads using the momentum signal
- does not alter the core economic market price

If you do not need this behavior yet, leave it disabled.

---

### Observability toggles

The current runtime also includes optional admin-facing visibility toggles.

```yaml
observability:
  admin-internal-status-enabled: false
  admin-benchmark-status-enabled: false
```

What these do:

| Key | Meaning |
| --- | --- |
| `admin-internal-status-enabled` | Adds queue, backlog, and backpressure details to `/dt status` |
| `admin-benchmark-status-enabled` | Adds benchmark-oriented internal timing output to `/dt status` |

Recommended posture:

- leave both off in normal operation
- enable them only during diagnostics or controlled validation

---

### Sounds

```yaml
sounds:
  enabled: true
  volume: 0.6
  ui-cooldown-ms: 120
  cycle-update-enabled: true
  pitch:
    buy-base: 1.2
    sell-base: 1.0
```

To disable sounds entirely:

```yaml
sounds:
  enabled: false
```

---

## `items.yml`

Every tradable item is defined under `items:`.

Minimum example:

```yaml
items:
  DIAMOND:
    display-name: "Diamond"
    base-price: 150.0
    category: MINING
```

Important fields:

| Field | Meaning |
| --- | --- |
| key such as `DIAMOND` | Minecraft material name in uppercase |
| `display-name` | Player-facing name |
| `description` | Optional lore text shown in the GUI |
| `base-price` | Long-term baseline price |
| `category` | Market group |
| `sigma` | Price sensitivity override |
| `vref` | Reference volume override |
| `inactive-gamma` | Idle recovery override |
| `idle-cycle-threshold` | Per-item stale threshold |
| `min-price-factor` | Floor multiplier relative to base price |
| `max-price-factor` | Ceiling multiplier relative to base price |

`display-name` is still supported and remains a valid editorial fallback. When the server language is Portuguese, DynaTrade now resolves item names in this order:

1. `plugins/DynaTrade/items_pt.yml`
2. bundled `items_pt.yml` inside the jar
3. `display-name` from `items.yml`
4. automatic formatting from the item key, such as `NETHERITE_INGOT -> Netherite Ingot`

This means you can keep `items.yml` focused on market tuning while using `items_pt.yml` only for Portuguese naming overrides.

---

## `items_pt.yml`

This file is optional and is only used for Portuguese item-name presentation.

Minimum example:

```yaml
version: 1

items:
  DIAMOND:
    display-name: "Diamante"
  IRON_INGOT:
    display-name: "Barra de Ferro"
```

Use this file when:

- you want Portuguese names without changing your main catalog
- you want to override the bundled default Portuguese translations
- you want admin-controlled naming for the GUI, `/price`, and trade receipts

Do not use this file to change item keys. Internal item identity remains the material key from `items.yml`.

---

## Recommended starting point

For most servers:

```yaml
language: en

economy:
  template: BALANCED

pricing:
  buy-spread: 0.06
  sell-spread: 0.10
  player-aware-pressure:
    target-participation-players: 4
    min-participation-factor: 0.25
    active-participation-reach:
      enabled: true
      min-active-reach-factor: 0.50
      active-participant-floor: 4
      active-participant-cap: 30
      reach-weight: 0.25

apply:
  max-per-tick: 8
  drain-deadline-ms: 15
```

Then tune categories and specific items only after observing real player behavior.

---

## Quick tuning guide

| Goal | What to adjust |
| --- | --- |
| Market feels too slow | Lower `vref`, raise `sigma`, or shorten cycle interval |
| Market feels too chaotic | Raise `vref`, lower `sigma`, or tighten price-factor caps |
| Rare items move too little | Raise per-item `sigma` and lower per-item `vref` |
| Common items crash too fast | Lower `sigma` and raise `vref` |
| Recovery feels too slow | Raise `inactive-gamma` or lower `idle-cycle-threshold` |
| Burst load feels rough on your server | Benchmark before changing `apply.*` defaults |

