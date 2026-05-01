# DynaTrade

DynaTrade is a batch-based economic engine for Paper 1.21.x servers.

Instead of reacting to every transaction, it collects player buy and sell activity over time and processes it in scheduled cycles. Prices update between cycles — not after every click. The result is a market that responds to real player behavior without constantly swinging.

---

## How it works

Players trade through the GUI or commands. Each trade is recorded as market pressure and held in a buffer. When a cycle runs, DynaTrade aggregates that pressure per item and runs it through a pricing pipeline: market pressure, volatility, mean reversion, and configurable price limits. The updated state is saved to disk immediately after.

On restart, the last confirmed state is restored. If the saved state is invalid, the plugin blocks startup rather than silently resetting prices.

---

## Requirements

- Paper 1.21.x (built and tested against 1.21.4)
- Java 21
- [Vault](https://www.spigotmc.org/resources/vault.34315/)
- A Vault-compatible economy provider — [EssentialsX](https://essentialsx.net/) is the tested option

Other Bukkit-family servers are not part of the current validated surface.

---

## Current version: 0.6.0

- Durable pending signal journal with stronger crash recovery behavior
- Serial async trade processing — disk I/O off the Bukkit thread
- Post-journal audit logging for trade failures after durability is confirmed
- Processing state feedback during buy and sell operations

---

For first-time setup, start with [Installation](Installation.md) and [Quick Start](Quick-Start.md).
For a full explanation of the pricing model, see [How the Market Works](How-the-Market-Works.md).
For day-to-day admin operations, see the [Admin Guide](Admin-Guide.md).

