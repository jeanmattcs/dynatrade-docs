# GUI Guide

This page documents the current DynaTrade GUI based on the in-game screens the project is using now.

Open the interface with `/market`.

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
- A back arrow in the lower-left navigation area
- A barrier close button in the lower-right navigation area

The listing is designed to let players browse quickly:

- Item icons are the first scan point
- The top row frames the page instead of wasting space on large decorative headers
- Navigation stays in the bottom row
- The middle rows are dedicated to the actual market inventory

---

## Item Preview

When a player highlights an item in the listing, the lore preview currently looks like this:

![Diamond Item Preview](https://i.imgur.com/j6zO97W.jpeg)

The current item preview format includes:

- Item name in gold
- Category name directly under the title
- A visual `MARKET` section divider
- Current price
- Buy and sell quote on one line
- Trend label
- Optional flavor description
- A bright `Click to trade` action hint

For the example shown:

- Item: `Diamond`
- Category: `Mining`
- Current price: `$180.00`
- Buy price: `$190.80`
- Sell price: `$162.00`
- Trend: `Stable`

This preview is doing the heavy lifting for readability. Players can understand the item before opening a deeper trade screen.

---

## Interaction Model

From the current screenshots, the GUI flow is:

1. Open `/market`
2. Choose a category or go into the full listing
3. Browse the listing grid
4. Hover an item to inspect the market preview
5. Click the item to continue into trading

Even when the player does not know exact material keys, the GUI gives enough context to browse visually and price-check naturally.

---

## Design Takeaways

The current GUI emphasizes:

- fast scanning over decoration
- strong color separation for important numbers
- icon-based category recognition
- clear bottom-row navigation
- practical item lore for market decisions

If future screenshots replace this layout, this page should be updated from those real screens rather than from older mock descriptions.
