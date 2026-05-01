# Commands and Permissions

This page is the complete reference for every DynaTrade command, its syntax, expected output, and the permission node that controls access.

---

## Player commands

These commands are available to all players by default. You can restrict them through your permission plugin.

---

### `/market`

Opens the DynaTrade market GUI.

**Permission:** `dynatrade.market` (default: `true`)

**What it does:**
- Opens the central market hub where players can browse categories, view items, and trade without memorizing item names.
- The GUI shows the current market price, buy price, sell price, and trend for each item.
- Trades made through the GUI use the same backend as the chat commands.

**Notes:**
- If the economy provider is unavailable, the GUI still opens but buy and sell actions are disabled.

---

### `/price <item>`

Shows the current quote for a single item.

**Permission:** `dynatrade.price` (default: `true`)

**Syntax:** `/price <MATERIAL_KEY>`

**Example:** `/price DIAMOND`

**Output:**
- Market price (the internal reference)
- Buy price (what the player would pay)
- Sell price (what the player would receive)

**Notes:**
- Item keys are Minecraft material names in uppercase (e.g. `IRON_INGOT`, `OAK_LOG`).
- This command never modifies the market. It is always safe to use for price lookups.

---

### `/buy <item> <quantity>`

Buys items from the market at the current buy price.

**Permission:** `dynatrade.buy` (default: `true`)

**Syntax:** `/buy <MATERIAL_KEY> <quantity>`

**Example:** `/buy DIAMOND 5`

**What it does:**
1. Validates the item exists in the catalog
2. Validates the requested quantity
3. Confirms the player has enough funds
4. Deducts the cost from the player's balance via Vault
5. Delivers the items to the player's inventory
6. Records buy pressure for the next market cycle

**Safeguards:**
- Rejects invalid or unknown items
- Rejects quantity of zero or negative
- Fails cleanly if the economy provider is unavailable
- Attempts rollback (refund) if item delivery fails after payment

---

### `/sell <item> <quantity>`

Sells items to the market at the current sell price.

**Permission:** `dynatrade.sell` (default: `true`)

**Syntax:** `/sell <MATERIAL_KEY> <quantity>`

**Example:** `/sell IRON_INGOT 64`

**What it does:**
1. Validates the item exists in the catalog
2. Validates the player holds enough of that item
3. Removes the items from the player's inventory
4. Deposits the proceeds into the player's balance via Vault
5. Records sell pressure for the next market cycle

**Safeguards:**
- Rejects invalid or unknown items
- Rejects quantity the player does not hold
- Attempts to restore removed items if the deposit fails

---

## Admin commands

All admin commands are grouped under `/dt`. The `dynatrade.admin` permission is required for most of them. The reset action requires the separate `dynatrade.admin.reset` permission.

---

### `/dt help`

Shows a summary of available `/dt` subcommands.

**Permission:** `dynatrade.admin`

---

### `/dt status`

Shows the current health of the DynaTrade runtime.

**Permission:** `dynatrade.admin`

**Output includes:**
- Economy provider status (connected or unavailable)
- Scheduler status (running or stopped)
- Time until the next scheduled cycle
- Current cycle generation number
- Number of tracked items
- Number of pending signals in the buffer
- Whether a prepared checkpoint is present
- Storage health summary

Use this command first when diagnosing any issue with the economy.

---

### `/dt item <item>`

Shows a focused diagnostic snapshot for one item.

**Permission:** `dynatrade.admin`

**Syntax:** `/dt item <MATERIAL_KEY>`

**Example:** `/dt item NETHERITE_INGOT`

**Output includes:**
- Current market price
- Buy price and sell price
- Simple trend label (`up`, `slight up`, `stable`, `slight down`, `down`, or `no recent movement`)
- Configured price floor and ceiling

Use this to quickly answer player reports about a specific item's pricing.

---

### `/dt reload`

Reloads configuration files and rebuilds the runtime.

**Permission:** `dynatrade.admin`

**What it does:**
1. Reloads `config.yml`, `items.yml`, and the active language file
2. Rebuilds the pricing runtime using the reloaded configuration
3. Restores the current market state from `market-state.yml`
4. Recovers any valid pending signals from the journal
5. Preserves the previous runtime if critical state is found to be invalid

**When to use:**
- After editing `config.yml` or `items.yml`
- After changing the language setting
- After adding or removing items from the catalog

**Notes:**
- In-memory prices are not reset by a reload. The persisted market state is restored as-is.
- If `market-state.yml` is invalid, reload will not silently apply base prices. It will log the problem and preserve the previous runtime.

---

### `/dt cycle`

Forces an immediate market cycle.

**Permission:** `dynatrade.admin`

**What it does:**
1. Drains the pending transaction buffer
2. Runs the pricing pipeline for all affected and idle items
3. Writes the cycle checkpoint to disk
4. Saves the updated market state
5. Applies the new prices to the live runtime
6. Cleans up the checkpoint and processed signals

**When to use:**
- During testing to see the effect of recent trades immediately
- After a controlled trade scenario to advance the market state
- When you want to process accumulated signals without waiting for the scheduled cycle

---

### `/dt reset`

Performs a full reset of the dynamic market state, restoring base prices from `items.yml`.

**Permission:** `dynatrade.admin.reset`

> ⚠️ This is a destructive operation. It cannot be undone. Use it only when you intend to wipe all dynamic price history and return the market to its configured starting state.

**How it works:**
1. The first call outputs a short confirmation token (5 characters) that expires in 60 seconds.
2. You must re-run `/dt reset <token>` within that window to confirm.
3. On confirmation, the scheduler is paused and the cycle lock is acquired.
4. In-memory buffers are cleared.
5. `pending-signals.yml`, `pending-signals.log`, and `cycle-checkpoint.yml` are deleted.
6. `market-state.yml` is rebuilt from the current `items.yml` base prices.
7. The runtime is restarted with a clean state.

**When to use:**
- After extensive test trading that you want to discard
- When recovering from a manually corrupted state that DynaTrade cannot read
- When restarting the economy from scratch intentionally

---

## Permissions reference

| Permission | Default | Controls |
|---|---|---|
| `dynatrade.market` | `true` | `/market` |
| `dynatrade.price` | `true` | `/price` |
| `dynatrade.buy` | `true` | `/buy` |
| `dynatrade.sell` | `true` | `/sell` |
| `dynatrade.admin` | `op` | `/dt status`, `/dt item`, `/dt reload`, `/dt cycle`, `/dt help` |
| `dynatrade.admin.reset` | `op` | `/dt reset` |

The `dynatrade.admin.reset` permission is intentionally separate from `dynatrade.admin`. This allows you to give trusted staff access to diagnostics and cycle control without allowing a full economy reset.

