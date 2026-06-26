# Testing

This page documents the validated test surface for DynaTrade `0.8.4`, the recorded validation material, and the expected behavior under extreme burst conditions.

---

## Test scope

DynaTrade was exercised across three broad areas:

- normal trade throughput under sustained synthetic load
- restart, reload, and persistence hardening
- burst behavior when the Bukkit main-thread apply queue becomes saturated

The goal was not just to confirm happy-path trades, but to verify durability, restart safety, and operator-facing behavior under pressure.

Additional validation for the current line also covered:

- player-aware pressure normalization as base pricing behavior
- active participation reach refinement for full-confidence cycles
- minimal effectiveVref calibration implemented and validated
- Market Insights dashboard validated through focused GUI tests covering summary rendering, top gainers, top losers, and most traded items
- runtime field validation with real player-backed trades through Paper + Mineflayer
- apply-before-signal trade ordering and journal-failure behavior
- pending delivery restart, partial-leftover, and concurrent-event behavior
- sell compensation, staged runtime retry recovery, pending Vault credit, degraded reload handling, item metadata policy, UUID lock cleanup, and delivery capacity

Latest automated baseline:

- `409` tests across `58` suites
- `0` failures and `0` errors
- clean compilation with the project still targeting Java `21`

---

## Validation posture

DynaTrade has recorded local load, race, recovery, and burst-behavior evidence for the `0.8.4` technical preview line.

Those artifacts are environment-dependent. Public documentation should treat them as operator validation material, not as fixed release-performance guarantees.

### Validated scenarios

| Load | Throughput | p50 | Success | Delta |
|---|---:|---:|---:|---:|
| Controlled nominal profile | recorded in validation materials | recorded in validation materials | see current evidence | see current evidence |
| Overload and burst profiles | recorded in validation materials | recorded in validation materials | environment-dependent | environment-dependent |

### What passed cleanly

- nominal load validation is documented separately in the validation set
- smoke validation for startup, trade flow, reload, restart, and fail-safe state blocking
- race benchmark with `50` concurrent bots and `500` total trades
- crash recovery benchmark with forced process kill and post-restart `delta = 0`
- reset benchmark covering `/dt reset`, state wipe, and post-reset trade viability
- audit-driven brutal-load validation with `100` bots / `1000` fire-and-forget commands, recovery OK, and post-drain `delta = 0`
- stale `IN_PROGRESS` recovery to non-retryable `MANUAL_REVIEW`
- partial delivery persistence without automatic full refund
- concurrent join/inventory-close events without duplicate delivery
- rejection at the `400` normal and `500` total delivery limits

---

## Burst behavior

The `50/1000` scenario is an overload test, not the normal operating profile of the plugin.

Under an extreme burst of `1000` trades in under `5` seconds, the apply queue drains on the Bukkit main thread using the configured `max-per-tick` budget. With `8` applies per tick and a `20 TPS` target, the queue can process roughly `160 trades/s` in the ideal case.

That means a full queue drain can take about:

`1000 trades / 8 per tick / 20 ticks per second = 6.25 seconds`

In practice, trades near the end of the queue may exceed the benchmark's `7s` client timeout window once normal overhead is included:

- command dispatch
- economy provider calls
- inventory mutation
- Bukkit tick jitter
- GC and local environment variance

### What this means

Observed degradation in the `50/1000` test is dominated by main-thread apply-path latency, not by journal fsync throughput.

In other words:

- the durability worker still commits batches quickly
- the apply queue becomes the bottleneck
- some late-queue trades time out from the benchmark's point of view before the success acknowledgment arrives

This defines a **throughput ceiling** for the current pipeline rather than a normal production profile.

---

## How to interpret `delta`

For any nominal validation scenario, `delta = 0` remains the primary correctness signal.

For overload scenarios, non-zero delta should be interpreted carefully:

- it shows that the cycle processed a different volume snapshot than the benchmark counted as acknowledged within its timeout window
- it does **not** automatically mean journal corruption
- it does indicate that burst latency exceeded the benchmark's acknowledgment SLA

This is why the `50/1000` result is useful as a ceiling test, but not as the baseline definition of normal server behavior.

For brutal fire-and-forget stress, DynaTrade is validated by the audit trail rather than by chat acknowledgments. The `100/1000` stress pass confirms that the applied runtime audit and processed cycle volume can reconcile to `delta = 0` after the queue is drained, while larger stages may run into server spam protection or client harness limits before they produce a clean throughput measurement.

---

## Pricing validation highlights

The current `0.8.4` line includes dedicated validation for the participation-aware pricing path:

- `1` player selling the same volume produces weaker adjusted pressure than multiple players selling that same total volume
- the dominant-side player count is direction-aware: buy pressure uses buyers, sell pressure uses sellers
- transactions without player identity preserve the old raw-pressure behavior
- recovery, replay, and pending payload restore remain volume-only and preserve the old fallback behavior
- active participation reach keeps `4/4` at full confidence but softens `4/30`

Recent runtime field validation also captured the full live path:

- one-player sell case: `sellUniquePlayers=1`, `participationFactor=0.25`
- four-player equal-volume sell case: `sellUniquePlayers=4`, stronger adjusted pressure than the one-player case
- next cycle reset confirmed that participation stays cycle-local
- volume-only startup restore confirmed `adjustedPressure == rawPressure`

## Minimal effectiveVref calibration validation

The current `0.8.4` line also includes focused validation for the minimal effectiveVref calibration.

Validated evidence in the current repository covers:

- disabled calibration fallback
- high-volume `effectiveVref` behavior
- low-volume `configuredVref` fallback
- EMA smoothing
- cap behavior
- restart-safe empty state
- config parsing and safe defaults
- focused tests plus a recorded full test-suite pass

The current public validation surface does not claim implementation of:

- `riskScore`
- volatility profiles
- thin-market detection
- `adaptiveSpreadBonus`
- admin calibration diagnostics

## Recommendations

- Normal operation: no tuning is needed.
- High-volume servers: consider increasing `apply.max-per-tick` to `12` after staged validation.
- Extreme burst events: temporary confirmation latency is expected while the apply queue drains.
- Long-term scaling: use the current `trade.admission.*` settings and validate any changes under load before widening the limits.

---

## Release position

The current `0.8.4` line is documented with:

- recorded local validation material
- stable restart and recovery behavior in the documented runs
- known burst-latency limits under heavy apply-queue pressure
- automated trade-consistency coverage for delivery recovery, rollback, item policy, capacity, and UUID locks

Not yet proven by public field evidence:

- a real Paper crash matrix at every `pending-deliveries.yml` persistence boundary
- long-duration delivery persistence behavior on slow disks
- an admin resolution workflow for `MANUAL_REVIEW`

The known limit is burst latency under extreme apply-queue pressure, which is documented here rather than treated as a blocker for ordinary server operation.

