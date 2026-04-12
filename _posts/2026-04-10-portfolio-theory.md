---
layout: post
title: "Applying Portfolio Theory to Sports Betting Markets"
date: 2026-04-10
categories: research
tags: [sports-betting, quantitative-finance, kelly-criterion]
mathjax: true
excerpt: "A rigorous treatment of the Kelly Criterion, fractional Kelly, multi-bet portfolio theory, and their connections to mean-variance optimization, with practical constraints for sports betting deployment."
---

## Introduction

The question of how much to bet, given a known or estimated edge, is as consequential as the question of whether to bet at all. Incorrect sizing — whether too aggressive or too conservative — systematically destroys value. Bet too little and the geometric growth rate is suboptimal; bet too much and the bankroll collapses via volatility-induced drawdowns. The **Kelly Criterion** provides the theoretically optimal solution: the fraction of bankroll that maximizes the expected growth rate of wealth.

This article derives the Kelly Criterion from first principles using the expected log-utility framework, extends it to fractional Kelly and multi-bet portfolios, and then connects the framework to the Markowitz mean-variance optimization familiar to quantitative finance practitioners. We conclude with the practical constraints imposed by real-world sports betting markets and describe how these constraints are implemented in our optimization pipeline.

---

## 1. The Kelly Criterion: Derivation from First Principles

### 1.1 Setup and Objective

Let $W_t$ denote the bettor's bankroll at time $t$. The bettor places a bet on a binary outcome with win probability $p \in (0,1)$ and net odds $b > 0$ (profit per unit staked). Let $f \in [0, 1]$ denote the **fraction of bankroll** staked. The bankroll evolves as:

$$
W_{t+1} =
\begin{cases}
W_t (1 + f \cdot b) & \text{with probability } p \\
W_t (1 - f) & \text{with probability } 1 - p
\end{cases}
\tag{1}
$$

The objective is to choose $f$ to maximize the **expected logarithmic utility** of next-period wealth:

$$
\max_{f \in [0,1]} \mathbb{E}[\log W_{t+1}] = \max_{f \in [0,1]} \left\{ p \log(W_t(1 + fb)) + (1-p) \log(W_t(1-f)) \right\}
\tag{2}
$$

Since $W_t > 0$ is fixed at decision time, the $\log W_t$ terms are constants that do not affect the optimization. Defining the **expected log-growth rate** $G(f)$:

$$
G(f) = p \log(1 + fb) + (1-p) \log(1-f)
\tag{3}
$$

we seek $f^* = \arg\max_{f \in [0,1]} G(f)$.

### 1.2 First-Order Condition

Differentiating $G(f)$ with respect to $f$:

$$
\frac{dG}{df} = \frac{p \cdot b}{1 + fb} + \frac{(1-p) \cdot (-1)}{1-f} = \frac{pb}{1+fb} - \frac{1-p}{1-f}
\tag{4}
$$

Setting this equal to zero:

$$
\frac{pb}{1+fb} = \frac{1-p}{1-f}
\tag{5}
$$

Cross-multiplying:

$$
pb(1-f) = (1-p)(1+fb)
\tag{6}
$$

Expanding both sides:

$$
pb - pbf = (1-p) + fb(1-p)
\tag{7}
$$

Collecting terms in $f$ on the left:

$$
pb - (1-p) = fbp + fb(1-p) = fb(p + 1 - p) = fb
\tag{8}
$$

Solving for $f$:

$$
f^* = \frac{pb - (1-p)}{b}
\tag{9}
$$

This can be rewritten as:

$$
f^* = p - \frac{1-p}{b}
\tag{10}
$$

Equation (9) is the **Kelly formula**. The numerator $pb - (1-p)$ is precisely the expected value of the bet per unit staked (from Article 1, equation 27). Thus:

$$
f^* = \frac{EV}{b}
\tag{11}
$$

The optimal Kelly fraction is the expected value per unit staked divided by the net odds. For a fair-odds bet ($EV = 0$), the Kelly fraction is zero.

### 1.3 Second-Order Condition (Verification of Maximum)

The second derivative of $G(f)$ is:

$$
\frac{d^2 G}{df^2} = -\frac{pb^2}{(1+fb)^2} - \frac{1-p}{(1-f)^2}
\tag{12}
$$

Both terms are negative for all $f \in [0,1)$, so $d^2G/df^2 < 0$ everywhere in the interior of the feasible set. This confirms that $f^*$ is a global maximum of $G(f)$, and $G(f)$ is strictly concave in $f$. $\square$

### 1.4 Positivity Condition

From equation (9), $f^* > 0$ iff $pb - (1-p) > 0$ iff $pb > 1-p$ iff $p > 1/(b+1) = p_{\text{implied}}$. This is precisely the positive-EV condition derived in Article 1. The Kelly Criterion prescribes positive stake if and only if the bet has positive expected value, and zero stake otherwise.

### 1.5 Numerical Example

Consider a Home team at $M_H = -130$ (so $b = 100/130 \approx 0.7692$) with estimated true win probability $p = 0.60$. Starting bankroll $W_0 = \$1,000$:

$$
f^* = \frac{0.60 \times 0.7692 - 0.40}{0.7692} = \frac{0.4615 - 0.40}{0.7692} = \frac{0.0615}{0.7692} \approx 0.0800
\tag{13}
$$

The Kelly Criterion recommends staking 8.0% of the bankroll, or $\$80$ on this bet. The expected log-growth for this single bet is:

$$
G(f^*) = 0.60 \log(1 + 0.08 \times 0.7692) + 0.40 \log(1 - 0.08)
\tag{14}
$$

$$
= 0.60 \log(1.0615) + 0.40 \log(0.92) = 0.60 \times 0.05970 + 0.40 \times (-0.08338) \approx 0.02248
\tag{15}
$$

Compounding over 100 independent bets at this edge, the expected bankroll multiplier is $e^{100 \times 0.02248} \approx 9.47$ — approximately a 9.5x return, assuming constant edge and odds.

---

## 2. Fractional Kelly: Variance Reduction at Reduced Growth

### 2.1 Motivation

The full Kelly fraction maximizes long-run growth but imposes substantial drawdown risk in finite time horizons. This occurs because the Kelly-optimal strategy is derived under the assumption of maximizing asymptotic growth rate, which is a property of the infinite-horizon limit. In practice, bankrolls are finite and bettors have meaningful risk aversion over the medium term.

Define the **fractional Kelly** fraction as:

$$
f_\alpha = \alpha \cdot f^*
\tag{16}
$$

where $\alpha \in (0, 1]$ is the scaling parameter. We now derive the effect of $\alpha$ on both expected growth rate and variance.

### 2.2 Growth Rate Under Fractional Kelly

The expected log-growth rate under fractional Kelly is:

$$
G(\alpha f^*) = p \log(1 + \alpha f^* b) + (1-p) \log(1 - \alpha f^*)
\tag{17}
$$

For small $f^*$ (which holds when edge is small), expand $\log(1+x) \approx x - x^2/2 + \ldots$:

$$
G(\alpha f^*) \approx p (\alpha f^* b - \frac{\alpha^2 (f^*)^2 b^2}{2}) + (1-p)(-\alpha f^* - \frac{\alpha^2 (f^*)^2}{2})
\tag{18}
$$

$$
= \alpha f^* [pb - (1-p)] - \frac{\alpha^2 (f^*)^2}{2}[pb^2 + (1-p)]
\tag{19}
$$

$$
= \alpha \cdot f^* \cdot b \cdot f^* - \frac{\alpha^2 (f^*)^2}{2} \cdot \sigma^2
\tag{20}
$$

where we define $\sigma^2 = pb^2 + (1-p)$ as the second moment of the bet return. Using $f^* b = EV/1 \cdot b/(b) = EV$ (from equation 11, $f^* = EV/b$, so $f^* b = EV$):

$$
G(\alpha f^*) \approx \alpha \cdot (f^*)^2 \cdot b^2 / b - \frac{\alpha^2 (f^*)^2 \sigma^2}{2}
\tag{21}
$$

More precisely, the second-order Taylor expansion of $G$ around $f = 0$ gives:

$$
G(f) \approx f \cdot \mu - \frac{f^2}{2} \sigma^2
\tag{22}
$$

where $\mu = pb - (1-p)$ is the mean return and $\sigma^2 = pb^2 + (1-p)$ is the second moment. The Kelly fraction maximizes this: $f^* = \mu / \sigma^2$. The maximum growth rate is $G(f^*) \approx \mu^2 / (2\sigma^2)$.

At fraction $\alpha f^*$:

$$
G(\alpha f^*) \approx \alpha f^* \mu - \frac{\alpha^2 (f^*)^2 \sigma^2}{2} = \alpha \frac{\mu^2}{\sigma^2} - \frac{\alpha^2 \mu^2}{2 \sigma^2} = \frac{\mu^2}{\sigma^2} \left(\alpha - \frac{\alpha^2}{2}\right)
\tag{23}
$$

Compared to full Kelly ($\alpha = 1$): $G(f^*) \approx \mu^2 / (2\sigma^2)$. The ratio is:

$$
\frac{G(\alpha f^*)}{G(f^*)} \approx 2\alpha - \alpha^2 = 1 - (1-\alpha)^2
\tag{24}
$$

For $\alpha = 0.5$: the growth rate ratio is $1 - 0.25 = 0.75$. Half-Kelly achieves 75% of the full-Kelly growth rate.

### 2.3 Variance Under Fractional Kelly

The variance of the log-wealth change per bet under fraction $f$ is:

$$
\text{Var}[\Delta \log W] = p (1-p) \cdot [\log(1+fb) - \log(1-f)]^2
\tag{25}
$$

For small $f$, using $\log(1+fb) \approx fb$ and $\log(1-f) \approx -f$:

$$
\text{Var}[\Delta \log W] \approx p(1-p)(fb + f)^2 = p(1-p) f^2 (b+1)^2
\tag{26}
$$

This is $O(f^2)$, and under fractional Kelly at $\alpha f^*$:

$$
\text{Var}[\Delta \log W] \approx \alpha^2 \cdot p(1-p)(f^*)^2(b+1)^2
\tag{27}
$$

Therefore, scaling from full Kelly to $\alpha f^*$:

- Expected growth rate scales as $\approx (2\alpha - \alpha^2) \approx \alpha$ for small $\alpha$ (linearly in $\alpha$)
- Variance scales as $\alpha^2$ (quadratically in $\alpha$)

At $\alpha = 0.5$: growth rate $\approx 75\%$ of full Kelly, variance $= 25\%$ of full Kelly. The variance reduction is disproportionately larger than the growth rate sacrifice — this is the central argument for fractional Kelly.

### 2.4 Optimal Alpha Under Risk Aversion

Given a risk-aversion parameter $\lambda > 0$ representing the bettor's marginal rate of substitution between expected growth and variance, the optimal $\alpha$ is found by maximizing the risk-adjusted growth rate:

$$
\max_\alpha \left[ G(\alpha f^*) - \lambda \cdot \text{Var}(\alpha f^*) \right]
\tag{28}
$$

Using the approximations above:

$$
\max_\alpha \left[ \frac{\mu^2}{\sigma^2}(\alpha - \frac{\alpha^2}{2}) - \lambda \cdot \alpha^2 p(1-p)(f^*)^2(b+1)^2 \right]
\tag{29}
$$

Taking the derivative with respect to $\alpha$ and setting to zero:

$$
\frac{\mu^2}{\sigma^2}(1 - \alpha) = 2\lambda \alpha \cdot p(1-p)(f^*)^2(b+1)^2
\tag{30}
$$

Solving for $\alpha^*$ yields an expression in terms of $\lambda$ and the bet parameters. In our platform, we use $\alpha = 0.25$ as the default, corresponding to quarter-Kelly, which provides substantial variance reduction while retaining meaningful growth rate.

---

## 3. Multi-Bet Kelly with Correlated Outcomes

### 3.1 The Multi-Dimensional Kelly Problem

In practice, a bettor places $N$ bets simultaneously across different games. Let $f_i \geq 0$ denote the fraction of bankroll allocated to bet $i$, and let $r_i$ denote the net return on bet $i$ (equal to $b_i$ if bet $i$ wins, $-1$ if it loses). The bankroll evolution is:

$$
W_{t+1} = W_t \left(1 + \sum_{i=1}^N f_i r_i\right)
\tag{31}
$$

The multi-dimensional Kelly problem is:

$$
\max_{\mathbf{f} \geq 0} \mathbb{E}\left[\log\left(1 + \sum_{i=1}^N f_i r_i\right)\right]
\tag{32}
$$

This is a concave maximization problem (since $\log$ is concave and $1 + \sum f_i r_i$ is linear in $\mathbf{f}$), so the maximum is unique when it exists.

### 3.2 The Independence Simplification

**Assumption (Independence):** The outcomes of different games are statistically independent, so $\mathbb{P}(r_i = b_i, r_j = b_j) = \mathbb{P}(r_i = b_i) \cdot \mathbb{P}(r_j = b_j)$ for $i \neq j$.

This assumption is appropriate for most MLB betting scenarios, where individual game outcomes are driven by on-field performance rather than correlated financial factors. (Exceptions include same-game parlays and doubleheader situations, which require explicit correlation modeling.)

Under independence, for small $f_i$ the expected log-wealth objective factorizes approximately:

$$
\mathbb{E}\left[\log\left(1 + \sum_{i=1}^N f_i r_i\right)\right] \approx \sum_{i=1}^N G_i(f_i) - \text{cross terms}
\tag{33}
$$

where $G_i(f_i) = p_i \log(1 + f_i b_i) + (1-p_i) \log(1 - f_i)$ is the single-bet Kelly objective for bet $i$.

**Claim:** Under independence, the unconstrained multi-dimensional Kelly solution is:

$$
f_i^* = \frac{p_i b_i - (1 - p_i)}{b_i}
\tag{34}
$$

applied independently to each bet.

**Proof sketch.** The gradient condition for the multi-dimensional problem is:

$$
\frac{\partial}{\partial f_i} \mathbb{E}[\log(1 + \sum_j f_j r_j)] = \mathbb{E}\left[\frac{r_i}{1 + \sum_j f_j r_j}\right] = 0
\tag{35}
$$

Under independence and for small positions, the denominator term $\sum_j f_j r_j$ is small relative to 1, so $1/(1 + \sum_j f_j r_j) \approx 1$ to first order. The gradient condition reduces to $\mathbb{E}[r_i] = 0$, which gives $p_i b_i - (1-p_i) = 0$, i.e., $f_i^* = EV_i / b_i$ as in the single-bet case. This approximation is exact when the betting fractions are infinitesimally small, and is an accurate approximation for the small fractions typical in practice (0.5%–5% per bet). $\square$

---

## 4. Portfolio Constraints: Motivation and Convexity

Real-world sports betting deployment requires constraints that go beyond the unconstrained Kelly optimum. We impose three primary constraints.

### 4.1 Daily Exposure Cap

Let $D \in (0, 1)$ be the maximum fraction of bankroll at risk across all simultaneous bets on a given day. The constraint is:

$$
\sum_{i=1}^N f_i \leq D
\tag{36}
$$

**Motivation:** Even under independence, the simultaneous loss of all bets on a day is a low-probability but non-negligible event. If $N = 15$ bets each have $f_i = 0.08$ and all lose simultaneously (probability approximately $0.45^{15} \approx 1.3 \times 10^{-5}$), the single-day loss is 120% of bankroll — a bankruptcy event. The daily cap ensures that even in extreme scenarios, daily drawdown is bounded by $D$.

### 4.2 Per-Game Cap

Let $G \in (0, 1)$ be the maximum fraction allocated to any single game:

$$
f_i \leq G \quad \forall i
\tag{37}
$$

**Motivation:** Individual model probability estimates have uncertainty (we address this explicitly in Article 6 via the variance-adjusted Kelly). The per-game cap limits exposure to model error on any single game. A typical value in our platform is $G = 0.04$ (4% of bankroll per game).

### 4.3 Same-Game Cap

For any game $k$ with both Home and Away sides potentially having positive edge, let $f_{H,k}$ and $f_{A,k}$ denote the corresponding fractions:

$$
f_{H,k} + f_{A,k} \leq S
\tag{38}
$$

**Motivation:** Betting both sides of the same game (a middle/arbitrage structure) is only profitable if the total stake is bounded. The same-game cap $S$ prevents the optimizer from constructing effectively risk-free arbitrage positions that would exhaust the bankroll in a single game.

### 4.4 Convexity of the Feasible Set

The feasible set $\mathcal{F} = \{\mathbf{f} \geq 0 : \sum_i f_i \leq D, f_i \leq G, f_{H,k} + f_{A,k} \leq S\}$ is the intersection of a non-negative orthant with finitely many linear inequality constraints. This is a convex polytope. Since the Kelly objective $G(\mathbf{f})$ is strictly concave (negative definite Hessian in the interior), the constrained maximization problem:

$$
\max_{\mathbf{f} \in \mathcal{F}} \sum_{i=1}^N G_i(f_i)
\tag{39}
$$

has a unique global optimum by the theory of convex optimization. The KKT conditions are necessary and sufficient, and can be solved efficiently via standard quadratic programming methods.

---

## 5. Kelly and Mean-Variance Optimization: The Connection

### 5.1 Second-Order Approximation

For small $f$, the log function admits a second-order Taylor expansion:

$$
\log(1 + f \cdot r) \approx f \cdot r - \frac{f^2 r^2}{2}
\tag{40}
$$

where $r$ is the random return on the bet. Taking expectations:

$$
\mathbb{E}[\log(1 + f \cdot r)] \approx f \cdot \mathbb{E}[r] - \frac{f^2}{2} \mathbb{E}[r^2]
\tag{41}
$$

For a portfolio of $N$ bets with fractions $\mathbf{f}$ and returns $\mathbf{r}$:

$$
\mathbb{E}\left[\log\left(1 + \sum_i f_i r_i\right)\right] \approx \sum_i f_i \mu_i - \frac{1}{2} \sum_{i,j} f_i f_j \sigma_{ij}
\tag{42}
$$

where $\mu_i = \mathbb{E}[r_i]$ and $\sigma_{ij} = \mathbb{E}[r_i r_j]$ (the second-moment matrix). Under zero correlation ($\sigma_{ij} = 0$ for $i \neq j$) and denoting $\sigma_i^2 = \mathbb{E}[r_i^2]$:

$$
\mathbb{E}\left[\log\left(1 + \sum_i f_i r_i\right)\right] \approx \sum_i f_i \mu_i - \frac{1}{2} \sum_i f_i^2 \sigma_i^2
\tag{43}
$$

### 5.2 The Mean-Variance Equivalence

The approximation in equation (43) is precisely the **Markowitz mean-variance objective** for a portfolio with mean $\mu_p = \sum_i f_i \mu_i$ and variance $\sigma_p^2 = \sum_i f_i^2 \sigma_i^2$ (under independence):

$$
\text{Kelly objective} \approx \mu_p - \frac{1}{2} \sigma_p^2
\tag{44}
$$

This is the mean-variance efficient frontier problem with risk-aversion coefficient $\gamma = 1$ (the coefficient implicit in the log-utility function). Practitioners familiar with Markowitz portfolio optimization can therefore interpret the Kelly Criterion as a mean-variance optimizer with a specific, theoretically grounded risk-aversion parameter.

This connection has an important practical implication: the Kelly framework is not a black box. Its outputs can be understood in the language of mean-variance optimization. A bet that the Kelly formula wants to overweight relative to its Sharpe ratio is one where the optimizer is exploiting the asymmetry in payoffs not captured by the second-order approximation — specifically, the left-tail protection from the log utility function.

### 5.3 Where the Approximation Fails

The mean-variance equivalence holds only to second order in $f$. For larger fractions — above roughly 10% of bankroll — the higher-order terms matter significantly. In particular, the log-utility objective strongly penalizes outcomes near $W = 0$ (since $\log(W) \to -\infty$ as $W \to 0^+$), while the mean-variance objective does not. This is why the Kelly Criterion provides finite ruin probability protection that a naive mean-variance optimizer cannot guarantee.

---

## 6. Practical Implementation: The Constrained Kelly Optimizer

### 6.1 Two-Stage Reallocation Algorithm

Our platform solves the constrained Kelly problem in two stages.

**Stage 1: Unconstrained Kelly Fractions.** For each bet $i$ with estimated probability $p_i$ and net odds $b_i$, compute the unconstrained Kelly fraction:

$$
f_i^{\text{raw}} = \max\left(0, \frac{p_i b_i - (1-p_i)}{b_i}\right)
\tag{45}
$$

The $\max(0, \cdot)$ operator enforces the no-short-selling constraint: we do not bet against games where the model finds no edge.

**Stage 2: Iterative Reallocation.** If any constraint in $\mathcal{F}$ is violated, the excess fraction is redistributed to other bets proportionally to their unconstrained fractions. This process iterates until all constraints are satisfied.

### 6.2 Why Iterative Reallocation is Necessary

A naive approach would simply clip each $f_i^{\text{raw}}$ at the per-game cap $G$ and then check the daily exposure constraint. However, this ignores the cross-constraint interactions: clipping one bet may free capacity under the daily cap, allowing other bets to be increased toward their unconstrained Kelly fractions.

The iterative reallocation algorithm ensures that the final allocation is the closest feasible point to the unconstrained Kelly solution in the sense of minimizing the shortfall in expected log-growth rate. Formally, each iteration solves a projection step onto the constraint hyperplane, and the sequence converges to the constrained optimum because the objective is strictly concave and the feasible set is closed and convex.

---

## Key Takeaways

The Kelly Criterion is derived by maximizing the expected logarithmic utility of next-period wealth. The resulting optimal fraction $f^* = (pb - (1-p))/b$ is positive if and only if the bet has positive expected value, and equals zero at the breakeven probability. The second-order conditions confirm this is a strict maximum, and the objective is globally concave, ensuring uniqueness.

Fractional Kelly at parameter $\alpha$ reduces expected growth rate approximately linearly in $\alpha$ while reducing variance quadratically — a favorable tradeoff. At half-Kelly, the bettor retains 75% of maximum growth while carrying only 25% of the variance. The optimal $\alpha$ depends on the bettor's risk-aversion parameter and can be derived analytically.

Extending to multiple simultaneous bets, the unconstrained Kelly solution under independence applies the single-bet formula independently to each wager. The Kelly objective admits a second-order approximation that connects directly to Markowitz mean-variance optimization with risk-aversion coefficient one, providing intuition for practitioners familiar with that framework. The approximation breaks down at large fractions, where the log-utility penalty near ruin dominates.

Practical deployment requires convex constraints on daily exposure, per-game exposure, and same-game exposure. These constraints form a convex polytope, and since the Kelly objective is strictly concave, the constrained optimum is unique and can be found efficiently. The two-stage iterative reallocation algorithm finds this optimum by projecting the unconstrained solution onto the feasible set while redistributing excess allocation to maximize realized growth rate.
