# Installation

This page covers every step required to install DynaTrade, verify that it is running correctly, and upgrade an existing installation.

---

## Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| **Paper** | 1.21.x | Built and validated against Paper API 1.21.4. Other Bukkit-family servers are not part of the current validated surface. |
| **Java** | 21 | Required by Paper 1.21.x. |
| **Vault** | Any current release | Required for economy integration. DynaTrade declares Vault as a soft dependency — the plugin loads without it, but buy and sell commands are disabled until Vault and an economy provider are present. |
| **Economy provider** | Any Vault-compatible | [EssentialsX](https://essentialsx.net/) is the recommended option. |

DynaTrade does **not** require a database. All market state is stored in YAML files inside the plugin folder.

---

## Fresh installation

### Step 1 — Obtain the jar

Use the pre-built artifact from the release page, or build from source:

```
mvn package
```

The output jar will be at `target/DynaTrade-<version>.jar`.

### Step 2 — Install Vault

Download Vault from [SpigotMC](https://www.spigotmc.org/resources/vault.34315/) and place it in your server's `plugins/` folder.

### Step 3 — Install an economy provider

Download and install [EssentialsX](https://essentialsx.net/) (or any other Vault-compatible economy plugin) into `plugins/`.

### Step 4 — Install DynaTrade

Copy `DynaTrade-<version>.jar` into `plugins/`.

Your `plugins/` folder should now contain at minimum:

```
plugins/
├── DynaTrade-<version>.jar
├── Vault.jar
└── EssentialsX-<version>.jar   (or your chosen economy provider)
```

### Step 5 — Start the server

Start the server once. DynaTrade will generate its configuration files:

```
plugins/DynaTrade/
├── config.yml
├── items.yml
└── languages/
    ├── messages_en.yml
    └── messages_pt.yml
```

### Step 6 — Verify the installation

Check the server console for these lines during startup:

```
[DynaTrade] [scheduler] started interval=<ticks>t (<seconds>s)
[DynaTrade] [runtime] commands registered.
[DynaTrade] [startup] plugin enabled version=<version> language=<lang> template=<template> templateOverrides=<n> economy=<ready|unavailable> restoredItems=<n> restoredSignals=<n>
```

Then in-game, run:

```
/dt status
```

A successful output looks like:

```
[DynaTrade] -- Status --
[DynaTrade] [SYS] Economy: OK
[DynaTrade] [SYS] Scheduler: active
[DynaTrade] [TIME] Next cycle: 04m 58s

[DynaTrade] [DATA] Generation: 0
[DynaTrade] [DATA] Items: 32
```

If Economy shows `Unavailable`, Vault or the economy provider is not loaded. See [Troubleshooting — Economy unavailable](Troubleshooting#economy-unavailable-after-startup).

### Step 7 — Review and adjust configuration

Open `plugins/DynaTrade/config.yml`. The defaults are safe for most servers. The minimum recommended review:

```yaml
language: en          # change to pt for Portuguese

economy:
  template: BALANCED  # STABLE | BALANCED | VOLATILE | HARDCORE
```

Open `plugins/DynaTrade/items.yml` to review the default item catalog. You can add or remove items at any time.

After any configuration change, run `/dt reload` — no restart required.

See [Configuration](Configuration.md) for a full reference.

---

## Upgrading from a previous version

DynaTrade persists market state between versions. Upgrading does not reset prices.

### Standard upgrade steps

1. Stop the server.
2. Replace the old `DynaTrade-<version>.jar` in `plugins/` with the new jar.
3. Do **not** delete `plugins/DynaTrade/`. Your configuration and market state live there.
4. Start the server.
5. Confirm the new version number in the console on startup.
6. Run `/dt status` to verify the runtime is healthy.

### Checking for configuration changes

New versions may add new keys to `config.yml` or `items.yml`. After an upgrade:

- Compare your existing `config.yml` against the newly generated default (if any).
- New optional keys will be missing from your file — DynaTrade will use internal defaults for any missing key, so nothing will break.
- If you want to use a new feature, add the corresponding key to your file manually.

### After a major version change

If the release notes mention breaking changes to `market-state.yml` or the item catalog format, follow the migration instructions in those release notes. Most upgrades do not require any migration.

---

## Uninstalling

To fully remove DynaTrade:

1. Stop the server.
2. Remove `DynaTrade-<version>.jar` from `plugins/`.
3. Optionally delete `plugins/DynaTrade/` if you no longer need the market data or configuration.

To temporarily disable DynaTrade without removing it, rename the jar (e.g. add `.disabled` as a suffix) — Paper will not load it on the next start.

---

## File layout reference

After installation and first startup, the complete plugin folder looks like:

```
plugins/DynaTrade/
├── config.yml                    ← global configuration — safe to edit
├── items.yml                     ← item catalog — safe to edit
├── languages/
│   ├── messages_en.yml           ← English text — safe to edit
│   └── messages_pt.yml           ← Portuguese text — safe to edit
├── market-state.yml              ← live market state — do not edit manually
├── pending-signals.yml           ← pending signal snapshot — do not edit manually
├── pending-signals.log           ← accepted trade journal — do not edit manually
└── cycle-checkpoint.yml          ← write-ahead checkpoint — do not edit manually
```

The files marked "do not edit manually" are managed exclusively by DynaTrade. Manual edits to those files risk corrupting the market state. See [Admin Guide — File reference](Admin-Guide#file-reference) for more details.

