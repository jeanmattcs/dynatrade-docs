# DynaTrade Wiki

DynaTrade is a batch-based economic engine for Minecraft servers running Paper 1.21.x.

It is not a traditional shop plugin. Instead of applying price changes after every transaction, DynaTrade collects buy and sell activity over time, processes it in scheduled market cycles, and applies structured price updates based on real player behavior.

The result is a market that feels alive without being chaotic - prices move, but they move with purpose.

---

## What DynaTrade does

- Tracks player buy and sell pressure in a real-time buffer
- Aggregates that pressure into scheduled pricing cycles
- Applies controlled price movement with configurable bounds per item and category
- Persists the resulting market state durably across restarts and reloads
- Recovers gracefully from crashes, partial writes, and corrupted artifacts
- Exposes prices through a GUI, chat commands, and admin diagnostics

## What DynaTrade does not do

- DynaTrade does not change prices after every click
- DynaTrade does not use a fixed price list
- DynaTrade does not require a database
- DynaTrade is not a player-to-player trading system

---

## Quick navigation

| Page | What it covers |
|---|---|
| [Quick Start](Quick-Start.md) | Installation, dependencies, first-time setup |
| [How the Market Works](How-the-Market-Works.md) | Cycle model, pricing pipeline, persistence |
| [Commands and Permissions](Commands-and-Permissions.md) | All commands, syntax, permission nodes |
| [Configuration](Configuration.md) | config.yml, items.yml, templates, tuning |
| [Admin Guide](Admin-Guide.md) | Day-to-day operations, diagnostics, logs |
| [Troubleshooting](Troubleshooting.md) | Common problems and how to resolve them |
| [Free vs Pro](Free-vs-Pro.md) | Feature scope comparison |

---

## Requirements

- Paper 1.21.x (built against Paper API 1.21.4)
- Java 21
- [Vault](https://www.spigotmc.org/resources/vault.34315/)
- A Vault-compatible economy provider (e.g. [EssentialsX](https://essentialsx.net/))

Other Bukkit-family servers are not part of the current validated surface.

---

## Current version

**0.6.0**

Highlights of this release:

- Durable pending signal journal with stronger crash recovery
- Serial async trade processing - disk I/O no longer runs on the Bukkit thread
- Post-journal audit logging for rare trade failures after durability is confirmed
- Processing state feedback during buy and sell operations

---

## Core concepts at a glance

**Market cycle** - The periodic event where DynaTrade aggregates pending trade pressure and recalculates item prices. Cycles run on a configurable tick interval. Players can trigger a manual cycle with `/dt cycle`.

**Three-price model** - Each item has three visible values: a market price (the internal reference), a buy price (what players pay), and a sell price (what players receive). The spread between them is configurable and creates healthy friction.

**Mean reversion** - Prices are pulled back toward their configured baseline over time. Items do not drift indefinitely.

**Persistence** - Market state is saved to disk after every cycle. DynaTrade is designed to recover the last confirmed market state after crashes or restarts.

**Fail-safe behavior** - If critical persisted data is unreadable or structurally invalid, DynaTrade refuses to silently reset the economy. It logs the problem and waits for admin action.

---

## Getting help

- Use [`/dt status`](Commands-and-Permissions#dt-status) to check the runtime health of the economy.
- Use [`/dt item <item>`](Commands-and-Permissions#dt-item) to inspect a specific item's current price, trend, and bounds.
- See [Troubleshooting](Troubleshooting.md) for common problems.
- See [Admin Guide](Admin-Guide.md) for operational procedures.

