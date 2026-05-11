# DynaTrade

DynaTrade is a batch-based market engine for Paper `1.21.x` servers.

Instead of reacting to every buy or sell like a slot machine, it aggregates player trade pressure, applies controlled price movement in cycles, and persists trade intent with recovery-focused durability. The result is a market that feels alive without turning ordinary server play into chaos.

---

## Current version: 0.7.0

Highlights of the current line:

- durable pending-trade journaling with stronger restart recovery
- async durability batching and cycle persistence off the Bukkit thread
- bounded main-thread apply backpressure for burst load
- processing-state feedback during buy and sell execution
- safer operator defaults for production servers

---

## How it works

High-level trade flow:

`/buy or /sell -> validation -> durable journal batch -> runtime apply -> transaction buffer -> pricing cycle`

What this means in practice:

- players still receive normal command and GUI feedback
- accepted trades are made durable before runtime side effects are considered complete
- the market still moves in cycles, not per click
- the Bukkit thread is protected from the worst disk and burst-apply pressure

On restart, the last confirmed market state is restored. If `market-state.yml` is critically invalid, DynaTrade blocks startup rather than silently resetting prices.

---

## Requirements

- Paper `1.21.x` (built and validated against `1.21.4`)
- Java `21`
- [Vault](https://www.spigotmc.org/resources/vault.34315/)
- A Vault-compatible economy provider - [EssentialsX](https://essentialsx.net/) is the tested option

Other Bukkit-family servers are not part of the current validated surface.

---

## Validated load profile

Recent validation on the current production runtime used:

- `50` bots
- `500` trades
- `500/500` succeeded
- `integrity.totalVolumeDelta = 0`

Recent valid runs landed around the `89-107 trades/s` range, with repeated confirmation runs averaging above `95/s`.

This is enough to treat the current `0.7.0` line as production-ready for the validated local profile rather than as an experimental milestone build.

---

For first-time setup, start with [Installation](Installation.md) and [Quick Start](Quick-Start.md).

For configuration and tuning, see [Configuration](Configuration.md).

For day-to-day operations, see the [Admin Guide](Admin-Guide.md).

