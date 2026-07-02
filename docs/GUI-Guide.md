# GUI Guide

This page documents the current DynaTrade GUI based on the in-game screens and flows the project is using now.

Open the interface with `/market`.

---

## Market Hub

The current `/market` flow starts at a hub screen rather than jumping straight into a single category or listing view.

What this hub exposes:

- `Explore` for the full market listing
- `Categories` for category-based browsing
- `Insights` for summary and ranking views
- `Statement` for the player's own recent trade history

This matters because `/market` is now the main navigation surface for both trading and lightweight market analysis.

---

## Current visual direction

The GUI uses a clean gray inventory frame with bright orange `DynaTrade` branding, white page subtitles, and simple icon-led navigation. The current screenshots show a practical, readable market UI rather than a decorative one.

---

## Category Selector

The current category selector looks like this:

![DynaTrade Categories](https://i.imgur.com/0MNJqOT.png)

What this screen shows:

- A 4-row inventory layout
- `DynaTrade - Categories` as the title
- A bookshelf header item centered on the top row
- Six category buttons across the second row
- A back arrow near the bottom center-left
- A barrier close button near the bottom center-right

The category buttons shown in the current UI are:

| Slot area | Icon | Category |
|---|---|---|
| Left | Iron Pickaxe | Mining |
| Left-center | Wheat | Agriculture |
| Center | Bundle | Miscellaneous |
| Right-center | Bricks | Building |
| Right | Blaze Rod | Magic |
| Far right | Glass Bottle | Alchemy |

Notes:

- The category selector is intentionally sparse and easy to scan.
- The icons are the primary visual cue for each market group.
- The bottom row is reserved for navigation rather than information overload.

---

## Market Listing

The item listing currently looks like this:

![DynaTrade Market Listing](https://i.imgur.com/QAgfmzS.jpeg)

What this screen shows:

- `DynaTrade - Market Listing` as the title
- A compass header item centered on the top row
- A dense item grid with many tradable entries visible at once
- Sort controls in the footer for changing how the listing is ordered
- A back arrow in the lower-left navigation area
- A barrier close button in the lower-right navigation area

The listing is designed to let players browse quickly:

- Item icons are the first scan point
- The top row frames the page instead of wasting space on large decorative headers
- Navigation stays in the bottom row
- The middle rows are dedicated to the actual market inventory
- Sort mode can change the order without changing the underlying market state

When the server language is Portuguese, the same listing automatically uses translated item names. Sorting follows the displayed name, so the visible item order can differ between English and Portuguese intentionally.

Current sort-oriented browsing can include:

- default listing order
- trend-oriented browsing
- volume-oriented browsing
- appreciation-oriented browsing
- depreciation-oriented browsing
- activity-oriented browsing

---

## Item Detail

When a player opens an item from the listing or an insights ranking, DynaTrade shows a dedicated item detail screen.

![Diamond Item Preview](https://i.imgur.com/j6zO97W.jpeg)

The current item detail format includes:

- Item name in gold
- Category name directly under the title
- Market price, buy price, and sell price
- Quick trade actions for fixed quantities such as `1`, `16`, and `64`
- Current price
- Trend label
- Confidence and coverage context when analytics data exists
- Recent-cycle context and compact visualization in the advanced view
- Optional flavor description

For the example shown:

- Item: `Diamond`
- Category: `Mining`
- Current price: `$180.00`
- Buy price: `$190.80`
- Sell price: `$162.00`
- Trend: `Stable`

Players can toggle between lighter and richer detail modes, and the richer mode is where the current public analytics surface becomes visible inside the trading flow.

---

## Player Statement

The hub also exposes a personal statement screen for the current player.

What it shows:

- recent buy and sell entries
- item, direction, quantity, quoted unit price, total value, timestamp, and cycle generation

This gives players a lightweight in-game trade receipt history without needing admin commands.

---

## Interaction Model

From the current screenshots, the GUI flow is:

1. Open `/market`
2. Choose Explore, Categories, Insights, or Statement
3. Browse a listing or ranking
4. Open an item detail screen when needed
5. Trade directly from the detail screen

Even when the player does not know exact material keys, the GUI gives enough context to browse visually and price-check naturally.

This is especially important now that translated item names can appear in the GUI while chat commands still expect English material keys. Players can browse naturally in the GUI, then use `/price`, `/buy`, or `/sell` with the material key if needed.

---

## Market Insights Dashboard

The Market Insights dashboard can be accessed from the `/market` hub. It provides a player-facing market summary screen showing:

- **Top Gainers** — items with the highest positive price movement
- **Top Losers** — items with the strongest negative price movement
- **Most Traded** — items with the highest buy + sell volume in the current cycle
- **Most Bought** — items with the highest buy-side volume
- **Most Sold** — items with the highest sell-side volume

Each section can lead into a dedicated ranking view. Those ranking views are paginated and can open the same item detail screen used by the rest of the market flow.

The dashboard is driven by the `MarketAnalyticsService` internal analytics read model and refreshes after each pricing cycle. It gives players visibility into market trends without requiring admin commands or external tools.

---

## Design Takeaways

The current GUI emphasizes:

- fast scanning over decoration
- strong color separation for important numbers
- icon-based category recognition
- clear bottom-row navigation
- practical item lore for market decisions

If future screenshots replace this layout, this page should be updated from those real screens rather than from older mock descriptions.
