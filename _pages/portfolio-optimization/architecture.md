---
layout: page
title: Architecture
permalink: /projects/portfolio-optimization/architecture/
---

The platform is organized into five discrete layers, each with
clearly defined inputs, outputs, and responsibilities. Raw data
is never modified — all derived outputs are fully recomputable
from the original records, and strategy experiments can be
replayed with different parameters without re-collecting data.

### Layer 1 — Data Acquisition

In plain terms: faithfully record what the market is saying at
each point in time, without computing anything.

Captures raw, timestamped market and source data from external
APIs. Every record carries a precise scrape timestamp, enabling
full reconstruction of the information set available at any
point in time. Outputs are written to immutable, append-only
database tables.

### Layer 2 — Normalization

In plain terms: ensure that records from different sources can
be reliably matched to the same game.

Creates a canonical, joinable representation of each game
across sources. Resolves identity — matching records from the
odds source, probability source, and scores source to a shared
game identifier. Silent mismatches at this layer corrupt all
downstream analysis, making robust canonical game matching the
single most important data integrity requirement in the system.

### Layer 3 — Feature Engineering

In plain terms: transform raw market data into the analytical
variables that drive betting decisions.

Derives the variables that strategies consume from normalized
data. All outputs are deterministic and fully recomputable.
Changing a parameter — such as the Bayesian confidence weight
— reruns this layer without touching raw data. Key outputs
include vig-removed fair market probability, Bayesian posterior
probability, expected value at each snapshot, line movement
features, and closing line value.

### Layer 4 — Strategy and Simulation

In plain terms: define a set of decision rules, apply them to
historical data, and measure what would have happened.

Applies parameterized decision rules to the feature dataset,
produces hypothetical bet selections, and simulates bankroll
trajectories through realized outcomes. A strategy is a fully
specified experiment: confidence weight, expected value
threshold, Kelly fraction, daily exposure cap, and capital
structure rules. A strict no-look-ahead constraint ensures that
a strategy evaluated at any point in time uses only information
that was available at that moment.

### Layer 5 — Evaluation

In plain terms: measure whether the strategy and its underlying
signal are actually working.

Produces diagnostics measuring signal quality and strategy
performance. Key outputs include bankroll trajectory, maximum
drawdown, probability calibration curves, predictive accuracy
by source and market segment, closing line value distribution,
and cross-strategy comparison. This layer is read-only relative
to all upstream layers.

### Sections

- [Framework](/projects/portfolio-optimization/framework/) — Theoretical foundations
- [Journal](/projects/portfolio-optimization/journal/) — Implementation log

