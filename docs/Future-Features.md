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
- expanded `/market` hub with Explore, Categories, Insights, and personal trade history
- Market Insights dashboard and ranking views showing market direction, top gainers, top losers, most traded, most bought, and most sold

## What Is Coming Next

### Historical market depth
The current runtime already keeps bounded multi-cycle analytics history internally and uses it for trend presentation. Future releases can expose more of that history to players and operators in broader public workflows.

### Broader trend workflows
The current runtime already computes trend signals and supports richer market browsing. Future work is about expanding how those signals are surfaced, explained, and operationalized rather than introducing trend awareness from scratch.

### Deeper per-item insight workflows
The current runtime already includes an item detail view with simple and advanced modes, recent-cycle context, and compact visualization. Future work can extend that into stronger long-window inspection and operator-facing analysis flows.

### Broader historical visualization
Future work can expand the current compact visual cues into broader history views, comparisons, and richer operator diagnostics without abandoning the lightweight in-game presentation.

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
