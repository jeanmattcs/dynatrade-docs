# DynaTrade

DynaTrade is a batch-based market engine for Spigot and Paper `1.21.x` servers.

Instead of reacting to every buy or sell like a slot machine, it aggregates player trade pressure, applies controlled price movement in cycles, and persists trade intent with recovery-focused durability. The result is a market that feels alive without turning ordinary server play into chaos.

---

## Current version: 0.8.2

Highlights of the current line:

- durable pending-trade journaling with stronger restart recovery
- apply-before-signal ordering, staged runtime retry recovery, and recovery-aware pending item delivery / Vault credit
- async durability batching and cycle persistence off the Bukkit thread
- bounded main-thread apply backpressure for burst load
- trade admission control for overload protection
- player-aware pressure normalization as base pricing behavior
- active participation reach for broader-economy participation context
- momentum signal generation
- optional momentum-driven quote adjustment, disabled by default
- processing-state feedback during buy and sell execution
- safer operator defaults for production servers

Current status:

- `Technical Preview`
- usable for controlled evaluation

---

## How it works

High-level trade flow:

`/buy or /sell -> validation/admission -> apply queue -> economy/inventory apply -> durable journal -> transaction buffer -> pricing cycle`

What this means in practice:

- players still receive normal command and GUI feedback
- market signals are made durable only after the real economy/inventory apply succeeds
- the market still moves in cycles, not per click
- the Bukkit thread is protected from the worst disk and burst-apply pressure

If apply succeeds but market journaling fails, the player effect remains and the price signal is omitted. This avoids recovery replaying pressure for a trade that never happened.

See [Trade Consistency and Recovery](Trade-Consistency-and-Recovery.md) for pending retry stages, pending delivery states, pending Vault credit, sell compensation, and operator procedures.

On restart, the last confirmed market state is restored. If `market-state.yml` is critically invalid, DynaTrade blocks startup rather than silently resetting prices.

---

## Requirements

- Spigot or Paper `1.21.x` (validated against `1.21.4`)
- Java `21`
- [Vault](https://www.spigotmc.org/resources/vault.34315/)
- A Vault-compatible economy provider - [EssentialsX](https://essentialsx.net/) is the tested option

The documented validated surface is `Spigot 1.21.4` and `Paper 1.21.4`.

---

## Validation position

The current `0.8.2` line has recorded local smoke, benchmark, and recovery evidence, but those artifacts are environment-specific and should be read from the current validation materials rather than from fixed public summary numbers.

This is enough to treat the line as a technical preview runtime for controlled evaluation, not as a broad public performance guarantee.

---

For first-time setup, start with [Installation](Installation.md) and [Quick Start](Quick-Start.md).

For configuration and tuning, see [Configuration](Configuration.md).

For day-to-day operations, see the [Admin Guide](Admin-Guide.md).

