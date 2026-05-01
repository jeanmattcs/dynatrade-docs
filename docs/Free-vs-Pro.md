# Free vs Pro

DynaTrade is available in two lines: Free and Pro.

The Free line includes the complete core economic engine. It is not a limited demo. Every core feature - the batch pricing cycle, mean reversion, persistence, crash recovery, the market GUI, and all admin commands - is part of Free.

Pro extends the engine with advanced market tools, richer analytics, and additional configuration surfaces. It is designed for servers that want deeper control and more visibility into market dynamics.

---

## Feature comparison

### Core engine

| Feature | Free | Pro |
|---|---|---|
| Batch-based market cycles | Yes | Yes |
| Dynamic prices driven by buy/sell pressure | Yes | Yes |
| Mean reversion toward configured base price | Yes | Yes |
| Idle recovery for stale prices | Yes | Yes |
| Max variation cap per cycle | Yes | Yes |
| Per-item price floor and ceiling | Yes | Yes |
| Economy templates (Stable, Balanced, Volatile, Hardcore) | Yes | Yes |
| Category-based tuning | Yes | Yes |
| Per-item sigma, vref, and behavior overrides | Yes | Yes |
| Fixed buy/sell spread | Yes | Yes |

### Player interface

| Feature | Free | Pro |
|---|---|---|
| Market GUI (`/market`) | Yes | Yes |
| Category browsing and paginated item listing | Yes | Yes |
| Item detail view with buy/sell actions | Yes | Yes |
| Price lookup command (`/price`) | Yes | Yes |
| Chat receipts and error feedback | Yes | Yes |
| Trade history / player statement panel | Planned | Yes |
| Advanced search and filtering in GUI | No | Yes |
| Per-item price trend history in GUI | No | Yes |
| Market pressure indicators | No | Yes |

### Admin tools

| Feature | Free | Pro |
|---|---|---|
| `/dt status` - runtime health check | Yes | Yes |
| `/dt item` - per-item diagnostic | Yes | Yes |
| `/dt reload` - safe hot reload | Yes | Yes |
| `/dt cycle` - manual cycle trigger | Yes | Yes |
| `/dt reset` - full market reset with confirmation | Yes | Yes |
| Per-cycle analytics and summaries | No | Yes |
| Admin GUI with market dashboard | No | Yes |
| Exportable audit log and trade trail | No | Yes |

### Persistence and storage

| Feature | Free | Pro |
|---|---|---|
| YAML-based market state persistence | Yes | Yes |
| Crash recovery with checkpoint and journal | Yes | Yes |
| Fail-safe behavior on corrupted state | Yes | Yes |
| Pending signal recovery across restarts | Yes | Yes |
| SQL-backed storage (optional) | No | Planned |
| Redis read cache / event channel | No | Planned |

### Pricing behavior

| Feature | Free | Pro |
|---|---|---|
| Standard fixed buy/sell spread | Yes | Yes |
| Dynamic spread based on market momentum | No | Planned |
| Premium pricing policy hooks | No | Planned |
| Per-category or per-item trade fees | No | Planned |

---

## What Free does not include

Free intentionally leaves out features that belong to a richer, more complex market layer:

- **Price trend history** - Free does not retain historical prices across cycles. Pro adds per-item time series storage.
- **Dynamic spread** - Free uses a fixed spread configured in `config.yml`. Pro can adjust the spread based on momentum.
- **Advanced admin GUI** - Free admins use chat commands for diagnostics. Pro includes a visual admin dashboard.
- **Rich analytics** - Free logs cycle summaries to the console. Pro exposes per-cycle and per-item analytics through the GUI and optionally exportable data.
- **SQL/Redis storage** - Free uses YAML files only. Pro can optionally use SQL as an authoritative store and Redis for caching and event distribution.

---

## What Pro is not

Pro is not a different engine. It shares the same pricing core, the same cycle model, and the same recovery guarantees as Free. Pro adds layers on top of a stable foundation - it does not replace it.

This means:

- Upgrading to Pro does not change how your economy behaves by default
- Pro features are additive - you opt into them through configuration
- The core market state format and recovery behavior remain the same

---

## Upgrade path

Upgrading from Free to Pro is designed to be safe and non-disruptive:

- Market state is preserved - prices, cycle generation, and history carry over
- No economy reset is required
- Pro features activate through configuration after the jar is swapped

Specific upgrade instructions will be included in the Pro distribution when it is released.

---

## Pro availability

The Pro line is currently in development. Features listed as "Planned" above are on the roadmap but not yet released.

If you need features beyond what Free provides, check back for Pro release announcements.

For most servers, **Free is the right place to start**. It delivers a complete, reliable, and well-tuned dynamic economy out of the box.
