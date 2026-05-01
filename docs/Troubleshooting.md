# Troubleshooting

This page covers the most common problems you may encounter with DynaTrade, what causes them, and how to resolve them safely.

---

## Economy unavailable after startup

**Symptoms:**
- `/buy` and `/sell` return a message saying the economy is unavailable
- `/dt status` shows `Economy: Unavailable`
- The market GUI opens, but buy and sell buttons are disabled

**Cause:**
DynaTrade could not find a Vault-compatible economy provider when it started.

**Resolution:**
1. Confirm Vault is installed and enabled (`/plugins`).
2. Confirm you have an economy provider installed (e.g. EssentialsX).
3. Check the server console for economy registration messages on startup.
4. Restart the server after installing the missing plugin.

DynaTrade does not require a restart after Vault and the economy provider are present together — but a restart is the safest way to ensure the integration initializes correctly.

---

## Prices are not changing

**Symptoms:**
- Players buy and sell items but prices stay the same
- `/price` always shows the same value

**Cause:**
Most likely one of these:
- The market cycle has not run yet
- The scheduler is stopped
- The trade volume is small relative to the configured `vref`

**Resolution:**
1. Run `/dt status` — check that the scheduler is `running` and note the next cycle time.
2. If the scheduler is stopped, run `/dt reload` to rebuild the runtime.
3. If the scheduler is running, wait for the next cycle or force one with `/dt cycle`.
4. If prices still do not change after a cycle, run `/dt item <item>` and check the price floor and ceiling. The item may already be at its limit.
5. If prices are moving but imperceptibly, consider lowering `vref` or increasing `sigma` for the affected category or item.

---

## Prices move too aggressively

**Symptoms:**
- A few trades cause very large price swings
- Common items become unaffordable or near-worthless quickly

**Cause:**
`sigma` is too high or `vref` is too low for the affected items or category.

**Resolution:**
1. Identify the affected items or category.
2. In `items.yml`, lower `sigma` and raise `vref` for the affected items — or apply the same change at category level in `config.yml` under `economy.overrides.category-behavior`.
3. Optionally lower `max-var-percent` to cap price movement per cycle.
4. Run `/dt reload` and monitor over several cycles.

---

## market-state.yml is invalid

**Symptoms:**
Console shows:
```
[DynaTrade] [startup] blocking startup because market-state.yml is invalid.
```
The economy is not running because the plugin enters fail-safe startup blocking.

**Cause:**
`market-state.yml` is structurally broken, has been manually edited incorrectly, or was truncated during a crash at an unusual time.

**Resolution (recovery path):**

Option A — Delete and reset (data loss):
1. Stop the server.
2. Delete or rename `plugins/DynaTrade/market-state.yml`.
3. Start the server.
4. DynaTrade will start with base prices from `items.yml` and persist a fresh market state on its next save cycle.

Option B — Full reset without data loss of other files:
1. Start the server without DynaTrade (remove the jar temporarily).
2. Inspect or restore `market-state.yml` from a backup.
3. Reinstall the jar and restart.

Option C — Use `/dt reset` (if the runtime partially started):
1. If DynaTrade is partially functional, run `/dt reset` and confirm the token.
2. This clears all dynamic state and rebuilds from base prices.

> DynaTrade intentionally does not silently reset the economy on an invalid state file. This protects you from losing market history due to an accidental edit.

---

## Recovery files were quarantined

**Symptoms:**
Console shows on startup:
```
[DynaTrade] [pending] pending signal payload is missing 'signals'; quarantining payload.
```

**Cause:**
An auxiliary recovery file was malformed — most likely from a hard crash during a write or an accidental manual edit.

**What this means:**
DynaTrade skipped the unreadable file and continued from the last confirmed clean `market-state.yml`. Some pending trade signals may have been lost, but the core market state is intact.

**Resolution:**
This is expected defensive behavior. No action is required unless you observe a significant unexpected price state after startup. If you do, compare current prices with the expected state and use `/dt reset` if a fresh start is preferable.

---

## Pending signals accumulating but not processing

**Symptoms:**
`/dt status` shows a growing pending signal count across multiple cycles with no change.

**Cause:**
- The scheduler stopped between cycles
- A cycle failed silently
- The market state could not be saved after a cycle ran

**Resolution:**
1. Check the console around the last expected cycle time for errors.
2. Run `/dt status` and confirm the scheduler is running.
3. If the scheduler is stopped, run `/dt reload`.
4. Force a cycle with `/dt cycle` and watch the console for errors during the cycle.
5. If saving fails (e.g. disk space issues), resolve the underlying I/O problem and reload.

---

## `/dt reload` does not apply new items

**Symptoms:**
You added an item to `items.yml`, ran `/dt reload`, but `/price <NEW_ITEM>` shows an error.

**Cause:**
Most likely one of:
- The item key is not a valid Minecraft material name
- The item key is lowercase
- The YAML structure is malformed (indentation error)

**Resolution:**
1. Confirm the item key uses uppercase Minecraft material names (e.g. `AMETHYST_SHARD`, not `amethyst_shard`).
2. Check `items.yml` YAML indentation — all items must be under the `items:` key at exactly one level of indentation.
3. Look for a parse error in the console after `/dt reload`.
4. Test the YAML in a YAML validator if you are unsure about the structure.

---

## Economy resets to base prices unexpectedly

**Symptoms:**
After a restart, all prices appear to be back to their base values from `items.yml`.

**Cause:**
Most likely `market-state.yml` was missing, empty, or unreadable at startup, and DynaTrade could not recover the previous state.

**Resolution:**
1. Check the startup console for messages about `market-state.yml`.
2. If DynaTrade logged a warn or severe about market state, the file was the problem.
3. Consider scheduling regular backups of `plugins/DynaTrade/market-state.yml`.

> Note: A missing `market-state.yml` at first startup is expected — DynaTrade creates it after the first cycle.

---

## Players see different prices in chat vs. GUI

**Symptoms:**
`/price DIAMOND` shows a different value than what the market GUI displays.

**Cause:**
This should not happen — the GUI and commands use the same trade backend and quote service. If you observe a mismatch:
- The GUI may still be showing a cached screen from before a recent cycle.
- Check if the player had the GUI open across a cycle boundary.

**Resolution:**
Ask the player to close and reopen `/market`. The GUI refreshes its price data when opened.

---

## Sounds playing too frequently or not at all

**Resolution:**
Adjust the `sounds` section in `config.yml`.

To disable sounds entirely:
```yaml
sounds:
  enabled: false
```

To reduce sound frequency:
```yaml
sounds:
  ui-cooldown-ms: 300
```

To disable cycle update sounds only:
```yaml
sounds:
  cycle-update-enabled: false
```

Run `/dt reload` after changes.

---

## General diagnostic checklist

When something seems wrong with the economy:

1. `/dt status` — is the economy running? Is the scheduler alive?
2. Check the console for `[WARN]` or `[SEVERE]` messages from DynaTrade.
3. `/dt item <item>` — is the item at its floor or ceiling?
4. Check `items.yml` — are `sigma`, `vref`, and price factors set reasonably?
5. Check `config.yml` — is the template correct? Are spreads configured as expected?
6. Force a cycle with `/dt cycle` and check the console cycle summary.
7. If nothing resolves the issue, run `/dt reload` to rebuild the runtime.
8. As a last resort, use `/dt reset` to start the economy from clean base prices.

