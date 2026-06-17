# Future Features

DynaTrade is already a functioning dynamic economy runtime for Spigot and Paper servers. The current system provides cycle-based pricing, player trading flows, market browsing, recovery-oriented persistence, and validation-backed operational behavior.

The areas below describe where the project is currently moving based on existing architecture documents, release planning, validation work, and feature reports. They are direction, not fixed commitments, and implementation details may evolve as testing and operational feedback continue.

The current focus remains on improving market interpretation, visibility, diagnostics, validation, and long-term extensibility while preserving operational safety.

The common thread across this work is continuity. DynaTrade is not moving from a concept into a product. It is evolving an already working market runtime toward a more explainable, better-instrumented, and more broadly usable system without giving up deterministic behavior, recovery safety, or operator clarity.

---

## Design Principles

Future development continues to follow the same principles that guide the current runtime:

- deterministic market behavior
- explainable pricing decisions
- operational visibility
- recovery-oriented design
- validation before expansion
- safe fallback behavior

New capabilities are expected to build upon these principles rather than replace them.

---

## Available Today

The current documented runtime already includes:

- dynamic item pricing driven by player buy and sell activity
- market buy and sell interactions through commands and GUI flows
- market GUI navigation with hub, category, listing, item detail, and player history views
- category-based item organization for browsing
- player-aware market pressure normalization in the base pricing model
- active participation reach for broader-economy context
- momentum-aware quote behavior in the spread layer
- operational diagnostics through the existing admin command surface
- recovery-oriented runtime design with pending signal, checkpoint, and persisted market state handling
- automated tests, runtime smoke coverage, field validation, and benchmark tooling

These capabilities establish the current baseline for future work. DynaTrade already treats pricing as a controlled cycle process rather than a click-by-click reaction system, already exposes a usable market surface to players, and already treats operational safety as part of the product rather than as a later concern.

---

## Market Calibration

One of the main areas of ongoing evolution is how the market interprets activity, not just how it stores prices. The current runtime already moved beyond pure volume by introducing player-aware pressure normalization and active participation reach. That work established an important principle: the market should react not only to how much volume appeared, but also to how credible and broadly supported that movement was.

The next step is to extend that same reasoning from participation into market scale and market maturity. Some items behave like common commodities, some behave like naturally thin markets, and some become unstable only under specific conditions. Future calibration work is intended to help DynaTrade distinguish those cases more reliably across time instead of relying only on static assumptions.

This is why documented future work in this area focuses on participation-aware interpretation, active reach as market context, adaptive market sensitivity, thin-market detection, volatility classification, and bounded risk analysis. These are not separate ideas competing for space. They are parts of the same goal: improving how the existing runtime judges significance while preserving explainability.

The long-term purpose of calibration is therefore conservative rather than dramatic. It is meant to make the market feel more credible, reduce disproportionate reactions, and give operators clearer reasons for what happened, without turning price movement into a hidden or self-contradictory system.

---

## Market Visibility

Market Visibility focuses primarily on understanding market activity at a broader system level. It is concerned with analytics, summaries, and the kinds of higher-level signals that help operators and advanced users understand what the market as a whole is doing.

The current market GUI already supports browsing items, viewing current prices, and opening item-level trading details. That gives players access to the market, but it still emphasizes isolated item views more than market-wide understanding.

Future visibility work grows naturally from that baseline. Once the runtime already tracks cycle results, price movement, trade history, and market state, the next useful step is to surface broader summaries such as top traded items, gainers, losers, and cycle-level movement in ways that are easier to read at a glance.

This matters because DynaTrade is intentionally a batch-based market. If prices move in cycles, both players and operators benefit from seeing what happened at the market level rather than only reading one item at a time. Better visibility is therefore not separate from the current design. It is the natural presentation layer for behavior the runtime already produces.

---

## Market Insights

Market Insights focuses on presenting that information to players through the existing market experience. It is concerned with how broader market context becomes visible inside the current GUI flow without forcing players into separate operator-facing tools.

In addition to item-by-item browsing, the market interface is expected to grow toward better summary views of what is happening across the economy. This direction builds directly on the current `/market` experience rather than introducing a disconnected parallel surface.

The goal of Market Insights is to make the current interface more informative. Instead of treating the GUI only as a place to open categories and select items, future work is intended to help players understand market direction, active items, and recent movement directly from the market itself.

This area matters because discoverability is part of usability. A dynamic market is easier to trust and use when players can see that the system has recognizable movement patterns, active areas, and broader context, not just a list of individual prices.

---

## Operational Visibility

DynaTrade places a strong emphasis on explainability. That affects not only pricing behavior, but also how operators inspect and troubleshoot the runtime. The current admin command surface already exposes operational state, and the current pricing pipeline already retains useful recent diagnostics such as raw versus adjusted behavior in the active model.

Future work extends that same principle. If the runtime becomes more sophisticated in how it interprets market activity, then operator tooling must become more capable of showing why a given movement occurred, what state the market was in, and whether a surprising outcome was expected, suspicious, or simply the result of ordinary market conditions.

This is why planned diagnostics focus on pricing explanation, item-level diagnostics, market diagnostics, and stronger troubleshooting support. They are not meant to turn DynaTrade into a monitoring platform. They are meant to preserve trust as the runtime grows more capable.

---

## Broader Item Ecosystems

The current runtime is still centered on the vanilla server item model, but existing planning already points toward a broader item-identity model over time. That direction follows from the same architectural principle already present elsewhere in the project: the market engine should depend on stable economic identity and runtime contracts, not on narrow assumptions that only hold for one catalogue shape forever.

This is why documented future work discusses namespaced identifiers, broader item abstraction, and compatibility with future custom or modded ecosystems. Examples of the identifier style under discussion include:

```text
minecraft:diamond
thermal:copper_ingot
create:brass_ingot
custom:admin_token
```

This is future direction, not a statement of current support. It matters because a market system becomes more useful when it can preserve the same economic model across a broader range of server ecosystems instead of being locked to a strictly vanilla identity model.

---

## Ecosystem & Integrations

Existing extensibility planning already treats interoperability as an important long-term concern, but without allowing external hooks to weaken the runtime core. That balance is consistent with the current architecture, which already isolates critical pricing and trade behavior carefully and treats optional extension layers as secondary to correctness.

Future integration work therefore matters less as a feature checklist and more as an architectural consequence. Once the runtime has stable internal boundaries, it becomes more practical to expose extension points, public events, and integration hooks for surrounding systems.

Examples may include:

- PlaceholderAPI
- custom plugins
- external services

The goal is to improve interoperability while preserving runtime stability, deterministic behavior, and operational clarity. In other words, ecosystem support should grow out of a stable core, not at the expense of it.

---

## Validation & Reliability

DynaTrade already treats validation as a first-class concern. The repository includes automated test coverage, focused recovery tests, runtime field validation, smoke runs on real server setups, and benchmark tooling. That existing posture is one of the clearest through-lines in the project.

Future work in this area builds directly on that baseline. As the runtime gains better market interpretation, broader diagnostics, and wider ecosystem ambitions, validation must continue to prove not only that features exist, but that behavior remains predictable under reload, restart, recovery, cycle isolation, live trading, and real server load.

This is why benchmarking, additional validation scenarios, runtime verification, operational tooling, and recovery validation remain central. For DynaTrade, reliability is not a final-stage concern. It is part of how new capabilities are accepted into the runtime at all.

---

## Long-Term Direction

The path toward `1.0` is not defined by a single feature or milestone.

Instead, it is defined by the gradual convergence of market interpretation, visibility, diagnostics, validation, and operational maturity.

The intended outcome is a market runtime that remains understandable to operators, useful to players, and dependable enough for long-term production use. That means stable dynamic pricing, explainable market behavior, actionable visibility, reliable operational tooling, broader item ecosystem support, and a deployment experience that feels mature rather than experimental.

Future work is therefore not about turning DynaTrade into a different kind of system. It is about carrying the current design principles forward until the runtime is not only feature-complete enough to be useful, but coherent enough to be trusted.

---

## Roadmap Notes

Future features represent current development direction derived from the project documentation, not fixed commitments.

Implementation order may change as validation results, architecture work, and operational feedback continue. Stability is preferred over rapid feature expansion, and backward compatibility, explainability, and operational clarity remain core project goals.
