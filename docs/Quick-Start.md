# Quick Start

This page walks you through installing DynaTrade and getting the market running for the first time.

---

## Requirements

Before installing DynaTrade, make sure you have:

- **Spigot or Paper 1.21.x** - DynaTrade is built against Spigot API `1.21.4-R0.1-SNAPSHOT` and validated on Spigot `1.21.4` and Paper `1.21.4`.
- **Java 21**
- **[Vault](https://www.spigotmc.org/resources/vault.34315/)** — required for economy integration.
- **A Vault-compatible economy provider** — [EssentialsX](https://essentialsx.net/) is the recommended option.

> DynaTrade will load without an economy provider, but buy and sell commands will be disabled until one is detected. The `/market` GUI and `/price` command will still work.

Current status:

- version: `0.8.4`
- public status: `Technical Preview`
- runtime status: usable for controlled evaluation

WARNING: Downgrade from 0.8.4 to 0.8.3 is not supported.
The rt3 audit log schema, PendingDeliveryService persistence format, and
architecture hardening state are incompatible with 0.8.3.
Downgrading may corrupt persisted state.

---

## Installation steps

### 1. Place the plugin jar

Copy `DynaTrade-<version>.jar` into your server's `plugins/` folder.

### 2. Install Vault

Download Vault and place it in `plugins/`. DynaTrade declares Vault as a soft dependency — it will not crash if Vault is absent, but trading will be unavailable.

### 3. Install an economy provider

EssentialsX is the recommended provider. Install it alongside Vault.

### 4. Start the server

Start the server once. DynaTrade will generate its configuration files inside:

```
plugins/DynaTrade/
├── config.yml
├── items.yml
├── items_pt.yml
├── languages/
│   ├── messages_en.yml
│   └── messages_pt.yml
```

### 5. Review the default configuration

Open `plugins/DynaTrade/config.yml`. The defaults are safe for most servers. At minimum, confirm:

```yaml
language: en          # or pt for Portuguese

economy:
  template: BALANCED  # recommended starting point
```

See [Configuration](Configuration.md) for a full explanation of every option.

### 6. Review the item catalog

Open `plugins/DynaTrade/items.yml`. A default item set is included. You can add, remove, or adjust items at any time and reload without restarting.

See [Configuration — items.yml](Configuration#itemsyml) for instructions on adding new items.

If your server uses Portuguese, open `plugins/DynaTrade/items_pt.yml` to customize how item names appear in the GUI, `/price`, and trade confirmations.

### 7. Apply changes

After editing any configuration file, run:

```
/dt reload
```

DynaTrade will reload config, drain in-flight trade work conservatively,
rebuild the runtime, and restore the persisted market state.

---

## First commands to try

Once the server is running and the economy provider is connected:

```
/dt status
```
Shows the current health of the economy engine, the scheduler state, and how many items are tracked.

If `/dt status` reports degraded mode instead of normal metrics, restart the
server before continuing. That means the last reload failed after the old
runtime had already been stopped.

```
/price DIAMOND
```
Shows the current market price, buy price, and sell price for Diamond.

Even if the server displays `Diamante` in Portuguese, the command still uses `DIAMOND`.

```
/market
```
Opens the market hub. This is the main player-facing interface and the entry point to:

- Explore: full item listing
- Categories: category-based browsing
- Insights: market summary and ranking views
- Statement: personal recent trade history

```
/buy DIAMOND 1
```
Buys one Diamond from the market at the current buy price.

```
/sell IRON_INGOT 10
```
Sells ten Iron Ingots to the market at the current sell price.

```
/dt cycle
```
Forces an immediate market cycle, processing all pending buy and sell pressure and updating prices. Useful during testing.

---

## Recommended first-session flow

1. Start the server and confirm the plugin enables cleanly in the console.
2. Run `/dt status` — confirm economy status is `OK` and the scheduler is running.
3. Run `/price DIAMOND` — confirm a price is shown with buy and sell values.
4. Open `/market` — confirm the hub opens, then test Explore, Categories, and Insights.
5. Run `/buy DIAMOND 1` — confirm the transaction completes and the receipt appears in chat.
6. Run `/sell IRON_INGOT 5` — confirm the sell completes.
7. Run `/dt cycle` — force a cycle and check the console for a cycle summary line.
8. Run `/price DIAMOND` again — confirm the price may have shifted based on the trade activity.

---

## What happens on restart

DynaTrade saves the market state at the end of every cycle and on server shutdown. When the server restarts, it loads the last saved state automatically. Prices continue from where they left off.

You do not need to re-run `/dt cycle` or `/dt reset` after a restart.

---

## Next steps

- [How the Market Works](How-the-Market-Works.md) — understand the cycle model and pricing logic
- [Configuration](Configuration.md) — tune templates, spreads, categories, and items
- [Admin Guide](Admin-Guide.md) — monitor the economy and respond to issues

