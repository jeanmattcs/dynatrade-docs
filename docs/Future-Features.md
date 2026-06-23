# Future Features

This page is intentionally high level.

It describes public direction for server owners, not the internal implementation backlog.

## Available Today

The current `0.8.3` technical preview already includes:

- cycle-based dynamic pricing
- commands and GUI trading
- durable journaling and restart recovery
- bounded apply backpressure
- player-aware pressure normalization and active participation reach
- minimal effectiveVref calibration in the current pricing runtime
- momentum signal generation and an optional quote-adjustment layer
- trade admission control

## Current Focus

The next public priority is stability and documentation alignment around the runtime that already exists today.

That means:

- clearer operator docs
- clearer validation boundaries
- cleaner separation between what is live now and what is still planned

## Planned Direction

Future work is expected to focus on:

- better market interpretation and calibration beyond the current minimal effectiveVref calibration
- stronger operator diagnostics
- broader market summaries and visibility
- more validation and release hardening
- broader item and ecosystem foundations over time

## Roadmap Note

Future direction is gate-driven, not calendar-driven.

Implementation order may change as validation, supportability, and documentation needs evolve.
