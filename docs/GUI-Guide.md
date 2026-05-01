# GUI Guide

The DynaTrade market GUI is the primary player-facing interface. It gives players a visual way to browse items, inspect live prices, and execute trades without needing to know item key names.

Open it with `/market`.

---

## Navigation overview

The GUI is organized across four screens. Navigation is always linear — you can go forward into detail or back to the previous screen. There are no dead ends.

```
/market
  └─ Central Market (Hub)
       ├─ Open Market ──► Market Listing (all items, paginated)
       │                       └─ [click item] ──► Item Detail (trade screen)
       │                                               └─ Back ──► Market Listing
       └─ Categories ──► Category Selector
                              └─ [click category] ──► Market Listing (filtered)
                                                           └─ [click item] ──► Item Detail
                                                                                   └─ Back ──► Market Listing (filtered)
```

Every screen has a **Back** button that returns to the previous screen, and a **Close** button (red barrier icon) that closes the GUI entirely.

---

## Screen 1 — Central Market (Hub)

**Title:** `DynaTrade — Central Market`
**Size:** 4 rows (36 slots)
**Open with:** `/market`

This is the entry point. It shows the main navigation options and a live countdown to the next market cycle.

### Layout

```
[ ][ ][ ][ ][H][ ][ ][ ][ ]   ← row 1: H = Clock header (next cycle countdown)
[ ][ ][ ][ ][E][ ][ ][ ][ ]   ← row 2: E = Compass (Open Market)
[ ][ ][ ][C][ ][S][ ][ ][ ]   ← row 3: C = Bookshelf (Categories), S = Book (Statement)
[ ][ ][ ][ ][X][ ][ ][ ][ ]   ← row 4: X = Barrier (Close)
```

### Buttons

| Slot | Icon | Label | Action |
|---|---|---|---|
| 4 | 🕐 Clock | DynaTrade | Info only. Shows the next cycle countdown and engine description. |
| 13 | 🧭 Compass | Open Market | Opens the full Market Listing with all tradable items. |
| 21 | 📚 Bookshelf | Categories | Opens the Category Selector. |
| 23 | 📖 Writable Book | Statement | Placeholder — player trade history. Coming in a future update. |
| 31 | 🚫 Barrier | Close | Closes the GUI. |

### The header (Clock item)

The Clock slot at the top center is an informational item. It shows:
- **Next cycle** — a live countdown (`MM:SS`) to the next scheduled price update
- **Economic engine** — a brief description of the cycle-based model

This counter updates in real time while the GUI is open.

---

## Screen 2 — Category Selector

**Title:** `DynaTrade — Categories`
**Size:** 4 rows (36 slots)

The Category Selector lets players filter the item listing by market group.

### Layout

```
[ ][ ][ ][ ][H][ ][ ][ ][ ]   ← row 1: H = Bookshelf header
[ ][M][A][C][ ][B][G][L][ ]   ← row 2: 6 category icons
[ ][ ][ ][ ][ ][ ][ ][ ][ ]   ← row 3: filler
[ ][ ][ ][←][ ][X][ ][ ][ ]   ← row 4: ← = Back, X = Close
```

### Category buttons (row 2)

| Slot | Icon | Category | Color |
|---|---|---|---|
| 10 | Iron Pickaxe | Mining | Gray |
| 11 | Wheat | Agriculture | Yellow |
| 12 | Bundle | Miscellaneous | Purple |
| 14 | Bricks | Building | Gold |
| 15 | Blaze Rod | Magic | Dark purple |
| 16 | Glass Bottle | Alchemy | Aqua |

Each button shows the number of items available in that category and opens a filtered Market Listing when clicked.

### Navigation buttons

| Slot | Icon | Action |
|---|---|---|
| 4 | Bookshelf | Info only — shows category count and next cycle countdown |
| 30 | Arrow | Back to Central Market |
| 32 | Barrier | Close the GUI |

---

## Screen 3 — Market Listing

**Title:** `DynaTrade — Market Listing` (or `DynaTrade — <Category>` when filtered)
**Size:** 6 rows (54 slots)

The Market Listing shows every tradable item, or all items in a selected category. Items are paginated — up to 36 items per page.

### Layout

```
[ ][ ][ ][ ][H][ ][ ][ ][ ]   ← row 1: H = Compass/Category header
[·][·][·][·][·][·][·][·][·]   ← rows 2-5: item grid (36 slots, 4 rows)
[·][·][·][·][·][·][·][·][·]
[·][·][·][·][·][·][·][·][·]
[·][·][·][·][·][·][·][·][·]
[←][ ][ ][‹][P][›][ ][ ][X]   ← row 6: navigation bar
```

### Header (row 1, slot 4)

When viewing all items: shows the Compass icon, total item count, and next cycle time.

When viewing a filtered category: shows the category name, item count, and next cycle time.

### Item grid (slots 9–44)

Each item slot shows:
- The item's Minecraft icon
- **Item name** (gold bold)
- Category label (color-coded by category)
- Market price, buy price, sell price
- Trend indicator (`▲ +4.2%`, `▼ −1.8%`, or `— Stable`)
- A short description from `items.yml` (if configured)
- `» Click to trade`

Click any item to open its detail screen.

### Navigation bar (row 6)

| Slot | Icon | Action |
|---|---|---|
| 45 | Arrow | Back (to Hub or Category Selector, depending on how you got here) |
| 48 | Arrow | Previous page |
| 49 | Paper | Current page indicator (Page X of Y) |
| 50 | Arrow | Next page |
| 53 | Barrier | Close |

Previous and Next only appear when the listing spans more than one page.

---

## Screen 4 — Item Detail (Trade Screen)

**Title:** `DynaTrade — Trading`
**Size:** 4 rows (36 slots)

The Item Detail screen is where trades happen. It shows the full price breakdown for a single item and provides buy and sell buttons at three quantity levels.

### Layout

```
[ ][ ][ ][ ][I][ ][ ][ ][ ]   ← row 1: I = Item icon with price lore
[ ][ ][ ][ ][ ][ ][ ][ ][ ]   ← row 2: filler
[ ][B1][B16][B64][ ][S1][S16][S64][ ]  ← row 3: buy and sell buttons
[ ][ ][ ][←][ ][X][ ][ ][ ]   ← row 4: ← = Back, X = Close
```

### Item header (slot 4)

The item icon at the top center shows:
- Category
- **Current price** (market reference)
- **Buy:** price · **Sell:** price
- **Trend** — movement since the last cycle
- **Next cycle** — live countdown

This information refreshes automatically when a cycle runs while the GUI is open.

### Buy buttons (slots 19, 20, 21) — Lime Dye

| Slot | Label | Action |
|---|---|---|
| 19 | Buy 1x | Buys 1 unit at the current buy price |
| 20 | Buy 16x | Buys 16 units at the current buy price |
| 21 | Buy 64x | Buys 64 units at the current buy price |

Each button shows the **total cost** before you click.

### Sell buttons (slots 23, 24, 25) — Red Dye

| Slot | Label | Action |
|---|---|---|
| 23 | Sell 1x | Sells 1 unit at the current sell price |
| 24 | Sell 16x | Sells 16 units at the current sell price |
| 25 | Sell 64x | Sells 64 units at the current sell price |

Each button shows the **total return** before you click.

### Navigation buttons

| Slot | Icon | Action |
|---|---|---|
| 30 | Arrow | Back to Market Listing |
| 32 | Barrier | Close |

### When the economy is unavailable

If Vault or the economy provider is not connected, all six trade button slots (19–21, 23–25) are replaced with a gray "Trading unavailable" item. The rest of the screen remains functional and readable.

### Price stale protection

When a market cycle runs while a player has the Item Detail screen open, the GUI detects that the displayed price is now outdated. If the player clicks a trade button after the price changed, DynaTrade refreshes the displayed price automatically before executing the trade. The player sees the updated numbers before confirming.

---

## Trend indicators

The trend label on item listings and the detail screen shows movement since the last completed cycle.

| Indicator | Meaning |
|---|---|
| `▲ +4.2%` (green) | Price rose in the last cycle |
| `▼ −1.8%` (red) | Price fell in the last cycle |
| `— Stable` (gray) | Movement was less than 0.05% — treated as stable |
| `— Stable` (gray, no data) | No cycle has run yet since this item was loaded |

The threshold for "stable" is intentionally small. A trend label only appears when there was meaningful movement.

---

## Cycle update sound

If a player has any DynaTrade GUI screen open when a market cycle runs, they hear a short audio cue. This indicates that prices may have changed. The listing and detail screens update their live data automatically.

The cycle update sound can be disabled in `config.yml`:

```yaml
sounds:
  cycle-update-enabled: false
```

