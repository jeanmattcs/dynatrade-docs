# Configuration

DynaTrade's configuration is split across two main files and an optional language file.

| File | Purpose |
|---|---|
| `plugins/DynaTrade/config.yml` | Global economy behavior, template selection, spreads, sounds, and debug flags |
| `plugins/DynaTrade/items.yml` | The item catalog — which items are tradable and how they behave |
| `plugins/DynaTrade/languages/messages_en.yml` | All player-facing and GUI text (English) |
| `plugins/DynaTrade/languages/messages_pt.yml` | All player-facing and GUI text (Portuguese) |

After editing any of these files, run `/dt reload` to apply changes without restarting the server.

> **Do not manually edit** `market-state.yml`, `pending-signals.yml`, `pending-signals.log`, or `cycle-checkpoint.yml`. These are managed exclusively by DynaTrade and hold live market data.

---

## config.yml

### Language

```yaml
language: en
```

Selects which language file is loaded from `plugins/DynaTrade/languages/`. Bundled options: `en`, `pt`.

---

### Economy template

```yaml
economy:
  template: BALANCED
```

The template is the simplest way to configure how the market should feel. It sets sensible defaults for volatility, mean reversion strength, reference volume, and cycle behavior.

| Template | Behavior |
|---|---|
| `STABLE` | Slow, predictable price movement. Good for survival or beginner-friendly servers. |
| `BALANCED` | Recommended default. Moderate price movement and reversion. |
| `VOLATILE` | Prices react more visibly to buy/sell pressure. Better for economy-focused servers. |
| `HARDCORE` | Aggressive price movement with wider swings. For competitive or high-stakes economies. |

Start with `BALANCED` if you are unsure.

---

### Economy overrides

Overrides let you customize specific parts of the selected template without replacing it entirely.

```yaml
economy:
  overrides:
    market: {}
    categories: {}
    category-behavior: {}
    pricing: {}
```

#### Pricing overrides

```yaml
economy:
  overrides:
    pricing:
      batch-interval-ticks: 6000
      sigma-default: 0.20
      gamma: 0.06
      vref-default: 120
      max-var-percent: 0.12
      min-price-factor: 0.25
      max-price-factor: 4.5
```

| Key | What it controls |
|---|---|
| `batch-interval-ticks` | Cycle frequency. 20 ticks = 1 second. `6000` = 5 minutes. |
| `sigma-default` | Default price sensitivity for items without a per-item override. |
| `gamma` | Mean reversion strength applied every cycle. |
| `vref-default` | Default reference volume. Lower = prices move more easily. |
| `max-var-percent` | Maximum price change per cycle. `0.12` = 12% cap. |
| `min-price-factor` | Global floor multiplier relative to each item's base price. |
| `max-price-factor` | Global ceiling multiplier relative to each item's base price. |

#### Category behavior overrides

Fine-tune an entire category without editing individual items.

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

Supported categories: `MINING`, `AGRICULTURE`, `MISCELLANEOUS`, `BUILDING`, `MAGIC`, `ALCHEMY`

| Key | What it controls |
|---|---|
| `sigma` | Price sensitivity for this category. |
| `vref` | Reference volume for this category. |
| `inactive-gamma` | Idle recovery strength for displaced items in this category. |
| `idle-cycle-threshold` | Quiet cycles before idle recovery accelerates. |

---

### Buy and sell spread

```yaml
pricing:
  buy-spread: 0.06
  sell-spread: 0.10
```

Controls the gap between the market price and the prices players see.

Example at market price 100:

```
buy-spread:  0.06  →  player pays  106
sell-spread: 0.10  →  player gets   90
```

Higher spread = more friction. Lower spread = players trade closer to the reference price.

---

### Idle recovery

```yaml
pricing:
  idle-cycle-threshold: 7
  idle-deviation-percent-threshold: 0.15
  idle-deviation-absolute-threshold: 0.20
  inactive-gamma: 0.20
```

Idle recovery pulls prices back toward their baseline when an item has been inactive for several cycles and remains displaced. See [How the Market Works — Mean reversion and idle recovery](How-the-Market-Works#mean-reversion-and-idle-recovery) for a full explanation.

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

To disable all sounds:

```yaml
sounds:
  enabled: false
```

`cycle-update-enabled` plays a subtle sound for players who have the market GUI open when a cycle updates.

---

### Debug logging

```yaml
pricing:
  batch-tick-log-enabled: false
  batch-empty-log-enabled: false
  batch-drain-summary-log-enabled: true
  batch-pricing-summary-log-enabled: true
```

Recommended production settings (as shown above): keep the summary logs on and the tick/empty logs off. Only enable verbose logging when actively diagnosing market behavior.

---

## items.yml

Every tradable item is defined under the `items:` key.

### Required fields

```yaml
items:
  DIAMOND:
    display-name: "Diamond"
    base-price: 150.0
    category: MINING
```

| Field | What it means |
|---|---|
| Key (e.g. `DIAMOND`) | Minecraft material name in uppercase |
| `display-name` | Name shown to players in the GUI and chat |
| `base-price` | The baseline price — the market starts here and reverts toward this |
| `category` | Market group: `MINING`, `AGRICULTURE`, `MISCELLANEOUS`, `BUILDING`, `MAGIC`, `ALCHEMY` |

### Optional fields

```yaml
description: "Text shown in the GUI item lore."
stock: -1
sigma: 0.5
vref: 50
inactive-gamma: 0.25
idle-cycle-threshold: 5
min-price-factor: 0.3
max-price-factor: 8.0
```

| Field | What it means |
|---|---|
| `description` | Short lore text shown in the market GUI |
| `stock` | Item availability. `-1` = unlimited. Reserved for future systems. |
| `sigma` | Per-item price sensitivity. Overrides category and global default. |
| `vref` | Per-item reference volume. Overrides category and global default. |
| `inactive-gamma` | Per-item idle recovery strength. |
| `idle-cycle-threshold` | Per-item quiet cycle count before idle recovery accelerates. |
| `min-price-factor` | Per-item price floor as a multiple of `base-price`. |
| `max-price-factor` | Per-item price ceiling as a multiple of `base-price`. |

Example — a rare speculative item:

```yaml
NETHERITE_INGOT:
  display-name: "Netherite Ingot"
  base-price: 500.0
  category: MINING
  description: "Forged in the Nether. High value, low liquidity."
  sigma: 0.8
  vref: 20
  min-price-factor: 0.4
  max-price-factor: 12.0
```

Example — a stable commodity:

```yaml
OAK_LOG:
  display-name: "Oak Log"
  base-price: 6.0
  category: BUILDING
  description: "A common building material."
  sigma: 0.15
  vref: 300
  min-price-factor: 0.5
  max-price-factor: 2.0
```

### Adding a new item

1. Open `plugins/DynaTrade/items.yml`
2. Add an entry under `items:` using a valid Minecraft material name (uppercase)
3. Set at minimum `display-name`, `base-price`, and `category`
4. Save the file
5. Run `/dt reload`
6. Confirm with `/price <ITEM_KEY>`

---

## Recommended starting configuration

For most servers, this is a safe and functional starting point:

```yaml
language: en

economy:
  template: BALANCED

pricing:
  buy-spread: 0.06
  sell-spread: 0.10
```

Add items to `items.yml`, test with `/price`, `/buy`, `/sell`, and `/dt cycle`, then tune categories and individual items based on observed market behavior.

---

## Configuration tuning guide

| Goal | What to adjust |
|---|---|
| Market feels too slow | Lower `batch-interval-ticks`, increase `sigma-default` or lower `vref-default` |
| Market feels too chaotic | Raise `vref-default`, lower `sigma-default`, or lower `max-var-percent` |
| Rare items move too little | Set higher `sigma` and lower `vref` per item |
| Common items crash too easily | Set lower `sigma` and higher `vref` per item, or tighten `max-price-factor` |
| Prices recover slowly after shocks | Increase `gamma` or lower `idle-cycle-threshold` |
| Prices recover too aggressively | Lower `gamma` and `inactive-gamma` |

