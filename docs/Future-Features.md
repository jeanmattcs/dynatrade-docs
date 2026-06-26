# Future Features

This page describes the direction DynaTrade is heading, not a commitment to specific dates or versions.

Implementation is gate-driven — each phase only begins when the current one is stable and well-documented enough to support it.

## Available Today

The current `0.8.4` technical preview already includes:

- cycle-based dynamic pricing driven by real player activity
- commands and GUI trading through `/market`, `/buy`, `/sell`, and `/price`
- durable trade journaling and restart recovery
- bounded apply backpressure for burst protection
- player-aware pressure normalization and active participation reach
- minimal effectiveVref calibration for high-volume items
- momentum signal generation and an optional quote-adjustment layer
- trade admission control for overload protection
- Market Insights dashboard: player-facing summary showing market direction, top gainers, top losers, and most traded items

## What Is Coming Next

### Historical market data
The current Market Insights dashboard shows only the latest cycle. Future releases will add bounded multi-cycle memory so players and admins can see how prices and rankings have moved over recent cycles instead of only the most recent one.

### Trend detection and browsing
Once historical data is available, the system will compute trend signals from recent cycles — distinguishing a sustained move from a one-cycle spike. Players will be able to browse the full market sorted by trend strength, volume, appreciation, or depreciation.

### Per-item insight screens
Each tradable item will gain a dedicated view showing its recent price path, volume history, predominant direction, and how current activity compares to its recent average.

### Compact historical visualization
Text and icon-based indicators will give players a quick visual sense of an item's recent trajectory — directional sequences, mini bar charts, or cycle-strip indicators that fit inside the existing GUI without adding clutter.

### Operator diagnostics
Administrators will get clearer visibility into why prices moved the way they did, what calibration decisions the runtime made, and how to investigate player reports without digging through raw files.

### Calibration improvements
Beyond the current minimal effectiveVref calibration, future work includes risk scoring, volatility profiles, thin-market detection, and adaptive spread behavior — making the market smarter about how it interprets trade activity across different items and conditions.

### Validation and hardening
Each new feature comes with dedicated validation. A dedicated hardening milestone will consolidate regression coverage, field validation, smoke tests, and benchmark evidence before the project moves toward a stable release.

### Broader item and ecosystem foundations
Longer-term work includes abstracting item identity beyond Bukkit Material, and preparing safe public API boundaries for other plugins to interact with DynaTrade's market data.

## Roadmap Note

None of these are calendar commitments. Each phase opens only when the current runtime is stable, documented, and validated enough to support it. The goal is a reliable market engine, not a race to add features.
