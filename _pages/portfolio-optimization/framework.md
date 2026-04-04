---
layout: page
title: Framework
permalink: /projects/portfolio-optimization/framework/
---

The platform is grounded in three interconnected bodies of
theory, each addressing a distinct dimension of the capital
allocation problem.

### Kelly Criterion

In plain terms: bet more when your edge is large, bet less when
it is small, and never bet so much that a losing streak can
wipe you out.

The Kelly criterion defines the optimal fraction of capital to
wager on a bet with positive expected value, maximizing the
long-run growth rate of wealth. For a moneyline bet with win
probability *p* and decimal payout *b* per unit wagered, the
optimal fraction is:

f* = (bp - q) / b

where *q = 1 - p*. A key practical insight is that overbetting
— wagering more than f* — is penalized far more severely than
underbetting. A bet sized at twice the Kelly fraction will grow
more slowly than one sized at half Kelly, and beyond a critical
threshold, overbetting leads to ruin even when the underlying
edge is real. In practice this means operating with a fractional
Kelly approach, typically between 0.25x and 0.5x the full Kelly
fraction, sacrificing a modest amount of growth rate in exchange
for significantly reduced drawdown risk.

### Bayesian Probability Adjustment

In plain terms: no external probability source should be trusted
completely. The market itself is a strong benchmark, and any
external estimate should be weighted against it in proportion
to how much it has proven to add.

The platform treats published win probability estimates as noisy
observations of the true win probability rather than ground
truth. The vig-removed market implied probability — derived by
normalizing the sportsbook's offered odds to remove the embedded
margin — serves as the prior. The source probability is then
blended against this prior to produce a posterior estimate:

p_posterior = w · p_source + (1 - w) · p_market

The confidence weight *w* reflects how much additional
information the source contributes beyond what the market
already knows. It is initialized conservatively and updated
over time using exponentially weighted scoring, comparing the
predictive accuracy of the source against the market on settled
outcomes. This mechanism ensures that the platform does not
overcommit to a probability source before its value has been
demonstrated empirically.

### Portfolio Optimization

In plain terms: when multiple opportunities exist simultaneously,
sizing each one in isolation ignores how they interact. The
correct approach treats the full set of bets as a portfolio and
allocates capital across all of them at once.

On any given day, multiple games may present positive expected
value simultaneously. Sizing each bet independently using the
single-bet Kelly formula ignores the interaction between
positions and systematically overestimates the optimal
allocation. The correct formulation maximizes expected log
wealth across the full slate jointly:

max E[log(1 + Σ fᵢ · Xᵢ)]

where fᵢ is the fraction of capital allocated to bet i and Xᵢ
is the random payoff. This is solved numerically, producing a
set of allocations that account for the full portfolio context.
For approximately independent moneyline bets with small edges,
a proportionally scaled approximation provides a fast and
accurate baseline, with the joint optimizer applied as the
primary sizing engine.

### Sections

- [Architecture](/projects/portfolio-optimization/architecture/) — System design
- [Journal](/projects/portfolio-optimization/journal/) — Implementation log

