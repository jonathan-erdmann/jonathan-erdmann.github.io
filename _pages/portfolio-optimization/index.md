---
layout: page
title: Portfolio Optimization Research Platform
permalink: /projects/portfolio-optimization/
---

A quantitative research platform for capital allocation under
uncertainty, built on sports betting markets as a transparent
and outcome-measurable proving ground.

Sports betting markets offer a rare combination of properties
that make them well-suited for this kind of research: odds are
publicly observable, outcomes are unambiguous, the market is
liquid enough to be competitive, and the full decision cycle —
from probability estimation to capital allocation to realized
outcome — completes within hours. These properties make it
possible to validate a capital allocation framework against real
data with a speed and clarity that most financial markets cannot
match.

The platform applies three interconnected bodies of theory to
MLB moneyline markets. The Kelly criterion provides the
theoretical basis for optimal bet sizing under uncertainty,
maximizing the long-run growth rate of capital. Bayesian
probability adjustment treats published win probability
estimates as noisy observations rather than ground truth,
blending them with market-implied probabilities to produce
calibrated posterior estimates. Portfolio optimization extends
the single-bet Kelly framework to a simultaneous multi-bet
setting, solving for the allocation that maximizes expected log
wealth across an entire day's slate of games.

The platform is designed as a research tool first. The
immediate objective is not to place bets but to determine
whether publicly available probability sources contain
information not already embedded in market odds — the
foundational question on which any edge depends. Strategy
execution is deferred until that question is answered
rigorously.

The codebase is written in R with a SQLite database backend,
organized into five discrete layers: data acquisition,
normalization, feature engineering, strategy and simulation,
and evaluation.

### Sections

- [Framework](/projects/portfolio-optimization/framework/) — Theoretical foundations
- [Architecture](/projects/portfolio-optimization/architecture/) — System design
- [Journal](/projects/portfolio-optimization/journal/) — Implementation log

