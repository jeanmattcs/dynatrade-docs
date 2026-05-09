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

Do not manually edit `market-state.yml`, `pending-signals.yml`, `pending-signals.log`, or `cycle-checkpoint.yml`. Those files contain live runtime state managed by DynaTrade.

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

The `0.6.4` production line includes bounded main-thread apply draining for durable trades.

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
