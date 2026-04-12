---
layout: post
title: "Sports Betting as Binary Options: A Black-Scholes Framework for Pricing and Timing Bet Execution"
date: 2026-04-10
categories: research
tags: [sports-betting, quantitative-finance, kelly-criterion]
mathjax: true
excerpt: "A rigorous development of the analogy between sports betting and binary options pricing, including optimal stopping theory for bet timing, uncertainty-adjusted Kelly fractions, and line dispersion as an implied volatility surface."
---

## Introduction

The analogy between sports betting and financial derivatives is not merely rhetorical. A moneyline bet is structurally identical to a **binary cash-or-nothing option**: the holder pays a premium, receives a fixed payout if a specified event occurs, and receives nothing otherwise. This structural equivalence opens the door to applying the extensive mathematical machinery of options pricing theory — stochastic calculus, optimal stopping, implied volatility — to the sports betting domain.

This article develops this analogy rigorously. We derive the Black-Scholes price for a binary option and map its components to sports betting, model line movement as a stochastic process, formalize bet timing as an optimal stopping problem, derive uncertainty-adjusted Kelly fractions, and develop the concept of line dispersion as an implied volatility surface. We conclude with an honest assessment of where the analogy illuminates and where it misleads.

The audience for this article includes practitioners with graduate-level exposure to stochastic calculus and financial mathematics. We assume familiarity with Itô's lemma, Brownian motion, and the Black-Scholes framework.

---

## 1. The Binary Option Analogy

### 1.1 Black-Scholes Price for a Cash-or-Nothing Call

In the Black-Scholes framework, consider an underlying asset $S_t$ following geometric Brownian motion:

$$
dS_t = \mu S_t \, dt + \sigma S_t \, dW_t
\tag{1}
$$

where $\mu$ is the drift, $\sigma$ is the volatility, and $W_t$ is a standard Brownian motion under the physical measure $\mathbb{P}$. A **cash-or-nothing binary call** with strike $K$, expiry $T$, and payout $Q$ pays $Q$ at time $T$ if $S_T > K$, and $0$ otherwise.

Under the risk-neutral measure $\mathbb{Q}$ (where $dS_t = r S_t \, dt + \sigma S_t \, d\tilde{W}_t$ for risk-free rate $r$), the time-$t$ price of the binary call is:

$$
C_{\text{binary}}(S_t, t) = e^{-r(T-t)} Q \cdot \mathbb{Q}(S_T > K | S_t)
\tag{2}
$$

Under GBM, $\log(S_T / S_t) \sim \mathcal{N}\left((r - \frac{\sigma^2}{2})(T-t), \sigma^2(T-t)\right)$ under $\mathbb{Q}$. Therefore:

$$
\mathbb{Q}(S_T > K | S_t) = \mathbb{Q}\left(\log\frac{S_T}{S_t} > \log\frac{K}{S_t}\right) = \Phi(d_2)
\tag{3}
$$

where $\Phi$ is the standard normal CDF and:

$$
d_2 = \frac{\log(S_t/K) + (r - \sigma^2/2)(T-t)}{\sigma\sqrt{T-t}}
\tag{4}
$$

The Black-Scholes price is:

$$
C_{\text{binary}} = e^{-r(T-t)} Q \cdot \Phi(d_2)
\tag{5}
$$

### 1.2 Mapping to Sports Betting

The sports betting analog maps as follows:

| Black-Scholes Component | Sports Betting Analog |
|---|---|
| Underlying $S_t$ | Aggregate information/sentiment about team quality |
| Strike $K$ | The threshold at which Home wins (trivially, $K$ = game played) |
| Binary payout $Q$ | Gross payout: stake + net profit = $c(1+b)$ |
| Premium paid | Stake $c$ (cost of the bet) |
| $\mathbb{Q}(S_T > K)$ | True win probability $p$ |
| $e^{-r(T-t)} Q \cdot \Phi(d_2)$ | Fair value of the bet: $p \cdot c(1+b) / (1+r(T-t))$ |
| Implied volatility $\sigma$ | Uncertainty in probability estimate |

The "risk-free rate" $r$ corresponds to the time value of money over the betting horizon, which is negligible for game horizons of hours to days. Setting $e^{-r(T-t)} \approx 1$:

$$
C_{\text{binary}} \approx Q \cdot p
\tag{6}
$$

A bet is fairly priced when the stake $c$ equals the fair value $Q \cdot p = c(1+b) \cdot p$, i.e., when $1 = (1+b)p$, i.e., when $p = 1/(1+b) = p_{\text{implied}}$. The bet is mispriced (and offers positive EV) when $c < Q \cdot p_{\text{true}}$, i.e., when $p_{\text{true}} > p_{\text{implied}}$.

The analogy clarifies that the bettor who identifies a mispriced binary option is in an analogous position to an options trader who identifies an option priced below its Black-Scholes fair value: both should buy the underpriced instrument, and the magnitude of the mispricing determines the expected profit.

---

## 2. Line Movement as a Stochastic Process

### 2.1 GBM Model of the Implied Probability

Let $L_t \in (0,1)$ denote the vig-removed implied probability at time $t$, where $t = 0$ is the game open and $t = T$ is game time. Because $L_t$ is bounded between 0 and 1, a geometric Brownian motion model for $L_t$ directly is inappropriate (GBM can exceed 1). We instead model the log-odds $\lambda_t = \log(L_t / (1 - L_t))$ as an arithmetic Brownian motion:

$$
d\lambda_t = \mu_\lambda \, dt + \sigma_\lambda \, dW_t
\tag{7}
$$

where $\mu_\lambda$ is the drift of the log-odds (representing the flow of information that systematically moves the market) and $\sigma_\lambda$ is the volatility. By Itô's lemma, $L_t = 1/(1+e^{-\lambda_t})$ satisfies:

$$
dL_t = L_t(1-L_t)\left[\mu_\lambda \, dt + \sigma_\lambda \, dW_t\right] + \frac{1}{2}L_t(1-L_t)(1-2L_t)\sigma_\lambda^2 \, dt
\tag{8}
$$

For near-even-money games ($L_t \approx 0.5$), the term $(1-2L_t) \approx 0$ and the Itô correction vanishes. The dynamics simplify to:

$$
dL_t \approx L_t(1-L_t)\left[\mu_\lambda \, dt + \sigma_\lambda \, dW_t\right]
\tag{9}
$$

This is approximately a GBM for $L_t$ with instantaneous standard deviation $\sigma_L = L_t(1-L_t)\sigma_\lambda$.

### 2.2 Distribution of $L_T$ Given $L_t$

Under the log-odds model, $\lambda_T | \lambda_t \sim \mathcal{N}(\lambda_t + \mu_\lambda (T-t), \sigma_\lambda^2(T-t))$. Transforming back to probability space, the distribution of $L_T | L_t$ is a **logit-normal distribution**:

$$
L_T | L_t \sim \text{LogitNormal}\left(\lambda_t + \mu_\lambda(T-t), \sigma_\lambda^2(T-t)\right)
\tag{10}
$$

For market efficiency, we expect the drift $\mu_\lambda$ to be approximately zero on average (prices contain no predictable directional bias after conditioning on available information). However, empirically there is often a drift in certain conditions:

- **Sharp money effect:** Early in the week, lines drift from their opening value as sharp bettors place large positions, pushing prices toward efficient levels.
- **Public money effect:** Closer to game time, retail betting volume may push lines toward popular teams, creating a mild predictable drift in some markets.

### 2.3 Mean-Reversion Properties

Empirically, implied probabilities in sports betting markets exhibit mild mean-reversion: extreme opening probabilities (very heavy favorites or underdogs) tend to drift toward the center over the betting week as the market digests information. This is analogous to mean-reverting interest rate models (Vasicek/CIR). A mean-reverting extension of the log-odds model is:

$$
d\lambda_t = \kappa(\bar{\lambda} - \lambda_t) \, dt + \sigma_\lambda \, dW_t
\tag{11}
$$

where $\kappa > 0$ is the mean-reversion speed and $\bar{\lambda}$ is the long-run log-odds level. Under this model, $\lambda_t$ follows an Ornstein-Uhlenbeck process, and the distribution of $\lambda_T | \lambda_t$ is:

$$
\lambda_T | \lambda_t \sim \mathcal{N}\left(\bar{\lambda} + (\lambda_t - \bar{\lambda})e^{-\kappa(T-t)}, \frac{\sigma_\lambda^2}{2\kappa}(1 - e^{-2\kappa(T-t)})\right)
\tag{12}
$$

This has practical implications for optimal bet timing, as developed in Section 4.

---

## 3. CLV as Option Value

### 3.1 The Value of Betting Now Versus Waiting

Suppose a bettor has identified a game where the current fair probability $L_t$ is below the estimated true probability $p$, generating positive expected value. The bettor can either: (a) bet now at price $L_t$, or (b) wait until time $s > t$ and bet at price $L_s$. The value of each option depends on how the line is expected to move.

The **CLV from waiting** — the additional value obtained by waiting from $t$ to $T$ (game time) rather than betting at $t$ — is:

$$
V_{\text{wait}}(t) = \mathbb{E}[L_T - L_t | \mathcal{F}_t]
\tag{13}
$$

where $\mathcal{F}_t$ is the information available at time $t$.

**Under zero drift ($\mu_\lambda = 0$):** The log-odds process is a martingale, so $\mathbb{E}[\lambda_T | \mathcal{F}_t] = \lambda_t$. Transforming back: $\mathbb{E}[L_T | \mathcal{F}_t] \approx L_t$ for near-even-money markets (using Jensen's inequality correction, the exact expectation of a logit-normal differs from the logit of the mean, but the correction is second-order). Therefore:

$$
V_{\text{wait}}(t) \approx 0
\tag{14}
$$

Under zero drift, the expected CLV from waiting is zero. The line is equally likely to move in favor of or against the bet. There is no expected benefit from delaying bet placement when the market is a martingale.

**Under positive drift:** If $\mu_\lambda > 0$ (the market is expected to move the probability upward, e.g., for a Home favorite), waiting generates positive expected CLV but at the cost of risk: the actual price at time $s$ may be worse than $L_t$ due to the stochastic component. This tradeoff is formalized as an optimal stopping problem in Section 4.

---

## 4. Optimal Stopping: When to Place the Bet

### 4.1 The American Option Analog

The problem of choosing when to place a bet is mathematically equivalent to the **American option exercise problem**: the bettor holds the option to execute the bet at any time before game time $T$, and must decide when to exercise based on observed line movements.

Let $\Pi(t) = (p - L_t) / L_t$ denote the instantaneous EV per unit staked if the bet is placed at time $t$. The bettor wants to maximize $\mathbb{E}[\Pi(\tau)]$ over stopping times $\tau \in [0, T]$.

Under the martingale (zero-drift) model, the optimal stopping problem has the trivial solution: stop at any time (there is no expected benefit from waiting). The bettor should place the bet as soon as positive EV is identified.

Under mean-reversion (equation 11), there is a non-trivial optimal stopping structure. For a bet on a favorite ($L_t > 0.5$, $\lambda_t > 0$), mean-reversion implies that the line is expected to drift toward $\bar{\lambda} < \lambda_t$, reducing the implied probability and therefore increasing the available edge $p - L_t$. This creates an incentive to wait.

### 4.2 The Optimal Stopping Boundary

Define the value function $\mathcal{V}(l, t) = \sup_{\tau \geq t} \mathbb{E}[(p - L_\tau)/L_\tau | L_t = l]$. The optimal stopping region is $\mathcal{S} = \{(l,t): \mathcal{V}(l,t) = (p-l)/l\}$ — the region where it is optimal to bet immediately.

The continuation region $\mathcal{C} = \{(l,t): \mathcal{V}(l,t) > (p-l)/l\}$ consists of states where the expected gain from waiting exceeds the current edge. In the continuation region, $\mathcal{V}$ satisfies the **Black-Scholes PDE**:

$$
\frac{\partial \mathcal{V}}{\partial t} + \mu_L(l) \frac{\partial \mathcal{V}}{\partial l} + \frac{1}{2} \sigma_L^2(l) \frac{\partial^2 \mathcal{V}}{\partial l^2} = 0
\tag{15}
$$

where $\mu_L(l)$ and $\sigma_L(l)$ are the drift and diffusion of $L_t$ derived from equation (11). The optimal stopping boundary $l^*(t)$ separates $\mathcal{S}$ from $\mathcal{C}$ and satisfies the smooth-pasting conditions:

$$
\mathcal{V}(l^*(t), t) = \frac{p - l^*(t)}{l^*(t)}, \quad \frac{\partial \mathcal{V}}{\partial l}\bigg|_{l^*(t)} = \frac{\partial}{\partial l}\left[\frac{p-l}{l}\right]\bigg|_{l^*(t)} = -\frac{p}{(l^*(t))^2}
\tag{16}
$$

**Practical interpretation.** The optimal stopping boundary provides the minimum edge threshold at which to bet at each point in time: bet when $p - L_t > l^*(t) \cdot [\text{threshold term}]$, wait otherwise. Closer to game time ($t \to T$), the continuation region shrinks because there is less time for mean-reversion to further improve the odds. At expiry, the boundary collapses: any positive edge should be taken immediately.

---

## 5. Volatility of the Probability Estimate as a Kelly Adjustment

### 5.1 Model Uncertainty and the Adjusted Kelly Fraction

The standard Kelly Criterion assumes that the true win probability $p$ is known with certainty. In practice, our model produces an estimate $\hat{p}$ that is subject to estimation error. Let the true probability be:

$$
p = \hat{p} + \epsilon, \quad \epsilon \sim \mathcal{N}(0, \sigma_p^2)
\tag{17}
$$

where $\sigma_p^2 > 0$ is the **model uncertainty variance**. The bettor observes $\hat{p}$ but not $p$.

### 5.2 Deriving the Adjusted Kelly Fraction

The Kelly objective under uncertainty is:

$$
G(f) = \mathbb{E}_{p}\left[p \log(1 + fb) + (1-p)\log(1-f)\right]
\tag{18}
$$

where the outer expectation integrates over the distribution of $p$. Using the tower property:

$$
G(f) = \hat{p} \log(1+fb) + (1-\hat{p})\log(1-f) + \mathbb{E}[\epsilon \log(1+fb) - \epsilon \log(1-f)]
\tag{19}
$$

Since $\mathbb{E}[\epsilon] = 0$, the linear terms in $\epsilon$ vanish. To capture the second-order effects of uncertainty, expand the objective to second order in $\epsilon$:

$$
G(f) \approx \hat{p}\log(1+fb) + (1-\hat{p})\log(1-f) + \text{correction term}
\tag{20}
$$

The correction captures the curvature of the Kelly objective in $p$. Taking the second derivative of $G(f)$ with respect to $p$ (treating $f$ as fixed):

$$
\frac{\partial^2}{\partial p^2}[p\log(1+fb) + (1-p)\log(1-f)] = 0
\tag{21}
$$

The objective is linear in $p$, so there is no second-order correction from uncertainty in $p$ itself. However, uncertainty in $p$ does affect the *optimal* $f^*$: the uncertainty-aware bettor solves $\max_f \mathbb{E}_p[G(f)]$, but the optimal $f$ under the distribution of $p$ differs from the nominal $f^*(\hat{p})$ because the optimal fraction is a nonlinear function of $p$.

**Exact derivation of the adjustment.** The nominal Kelly fraction is $f^*(\hat{p}) = (\hat{p}b - (1-\hat{p}))/b$. The true Kelly fraction given $p = \hat{p} + \epsilon$ is:

$$
f^*(p) = f^*(\hat{p}) + \frac{df^*}{dp}\bigg|_{\hat{p}} \epsilon + \frac{1}{2}\frac{d^2f^*}{dp^2}\bigg|_{\hat{p}} \epsilon^2 + \ldots
\tag{22}
$$

Since $df^*/dp = (b + 1)/b = 1/p_{\text{implied}}$ (constant in $p$) and $d^2f^*/dp^2 = 0$, the true optimal fraction is linear in $p$. The expected true optimal fraction is:

$$
\mathbb{E}[f^*(p)] = f^*(\hat{p})
\tag{23}
$$

So naive use of the nominal fraction $f^*(\hat{p})$ is unbiased on average. However, the *value* generated by using $f^*(\hat{p})$ when the true optimal fraction differs by $\epsilon/p_{\text{implied}}$ involves a second-order loss from Kelly objective concavity:

$$
\mathbb{E}[G(f^*(\hat{p}), p)] \approx G(f^*(\hat{p}), \hat{p}) - \frac{1}{2} \left|\frac{d^2G}{df^2}\right|_{f^*} \cdot \left(\frac{df^*}{dp}\right)^2 \sigma_p^2
\tag{24}
$$

The second derivative of $G$ with respect to $f$ (from Article 2, equation 12) is:

$$
\frac{d^2G}{df^2} = -\frac{pb^2}{(1+fb)^2} - \frac{1-p}{(1-f)^2} \approx -\sigma_R^2 \text{ (for small } f)
\tag{25}
$$

The correction term is:

$$
\text{loss} \approx \frac{1}{2} \sigma_R^2 \cdot \left(\frac{1}{p_{\text{implied}}}\right)^2 \sigma_p^2
\tag{26}
$$

The **uncertainty-adjusted Kelly fraction** is derived by maximizing the adjusted objective:

$$
G_{\text{adj}}(f) = G(f, \hat{p}) - \frac{1}{2} \sigma_R^2 \left(\frac{f}{p_{\text{implied}}}\right)^2 \cdot \frac{\sigma_p^2}{\hat{p}(1-\hat{p})}
\tag{27}
$$

Taking the derivative and setting to zero gives an adjusted fraction:

$$
f^*_{\text{adj}} = f^* \cdot \left(1 - \frac{\sigma_p^2}{\hat{p}(1-\hat{p})}\right)
\tag{28}
$$

This result states that estimation uncertainty multiplicatively reduces the optimal Kelly fraction by the factor $1 - \sigma_p^2 / (\hat{p}(1-\hat{p}))$. The denominator $\hat{p}(1-\hat{p})$ is the maximum entropy (Bernoulli variance) at the estimated probability.

**Interpretation.** When $\sigma_p^2 \ll \hat{p}(1-\hat{p})$ (model uncertainty is small relative to the inherent randomness of the event), the adjustment is negligible and $f^*_{\text{adj}} \approx f^*$. When $\sigma_p^2 \to \hat{p}(1-\hat{p})$ (maximum uncertainty — we know nothing more than the base rate), the adjusted fraction goes to zero: a bettor who knows nothing should not bet. The formula is the Kelly analog of the Bayesian shrinkage estimator.

**Numerical example.** Suppose $\hat{p} = 0.58$, $b = 0.909$ (corresponding to $-110$ odds), and the model uncertainty is $\sigma_p = 0.03$ (standard deviation of 3 probability points). Then:

$$
\hat{p}(1-\hat{p}) = 0.58 \times 0.42 = 0.2436
\tag{29}
$$

$$
f^*_{\text{adj}} = f^* \left(1 - \frac{(0.03)^2}{0.2436}\right) = f^* \left(1 - \frac{0.0009}{0.2436}\right) = f^* \times 0.9963
\tag{30}
$$

For small $\sigma_p = 0.03$, the adjustment is minor (less than 0.4%). With $\sigma_p = 0.10$:

$$
f^*_{\text{adj}} = f^* \left(1 - \frac{0.01}{0.2436}\right) = f^* \times 0.9589
\tag{31}
$$

A 10 probability-point standard deviation in model estimates reduces the Kelly fraction by approximately 4%. The adjustment becomes significant when $\sigma_p \geq 0.1$.

---

## 6. Line Dispersion as an Implied Volatility Surface

### 6.1 Definition and Interpretation

In options markets, **implied volatility** is the value of $\sigma$ that equates the Black-Scholes model price with the observed market price. When different strikes or maturities imply different volatilities, the resulting surface is the **implied volatility surface** (volatility smile/skew).

The sports betting analog is **line dispersion** across bookmakers: the cross-sectional spread of fair probabilities offered by different books on the same game. Define:

$$
\sigma_{\text{implied}}^{(g)} = \text{std}\left(\{L_{t,k}^{(g)}\}_{k=1}^B\right)
\tag{32}
$$

where $L_{t,k}^{(g)}$ is the vig-removed fair probability for game $g$ from book $k$ at time $t$. High $\sigma_{\text{implied}}$ indicates that books disagree significantly on the true probability.

### 6.2 Dispersion-Adjusted Kelly Fraction

High line dispersion serves as a proxy for high uncertainty about the true probability — the analog of high implied volatility in options. Using the framework from Section 5:

$$
\sigma_p^2 \approx c \cdot (\sigma_{\text{implied}}^{(g)})^2
\tag{33}
$$

where $c > 0$ is a calibrated scaling constant. Substituting into equation (28):

$$
f^*_{\text{disp}} = f^* \cdot \left(1 - \frac{c \cdot (\sigma_{\text{implied}}^{(g)})^2}{\hat{p}(1-\hat{p})}\right)
\tag{34}
$$

Games with high cross-book dispersion receive reduced Kelly fractions, reflecting the uncertainty signal embedded in the dispersion. This is the operational implementation of the volatility-adjusted Kelly in our platform.

**Practical example.** For a game where most books price Home at fair $p \approx 0.58$ but there is high dispersion with $\sigma_{\text{implied}} = 0.02$, and the model estimates $\hat{p} = 0.62$:

$$
f^* = \frac{0.62 \times 0.909 - 0.38}{0.909} = \frac{0.5636 - 0.38}{0.909} = \frac{0.1836}{0.909} \approx 0.202
\tag{35}
$$

Applying the dispersion adjustment with $c = 2$ (empirically calibrated):

$$
f^*_{\text{disp}} = 0.202 \times \left(1 - \frac{2 \times (0.02)^2}{0.2436}\right) = 0.202 \times \left(1 - \frac{0.0008}{0.2436}\right) = 0.202 \times 0.9967 \approx 0.201
\tag{36}
$$

For this modest dispersion level, the adjustment is minor. High-dispersion games with $\sigma_{\text{implied}} = 0.05$:

$$
f^*_{\text{disp}} = 0.202 \times \left(1 - \frac{2 \times (0.05)^2}{0.2436}\right) = 0.202 \times \left(1 - 0.0206\right) = 0.202 \times 0.979 \approx 0.198
\tag{37}
$$

---

## 7. Limitations of the Black-Scholes Analogy

The Black-Scholes framework provides powerful intuition for sports betting, but the analogy has critical limits that must be acknowledged.

**No continuous hedging.** The central mechanism of Black-Scholes — dynamic delta hedging — is unavailable in sports betting. A derivative trader can continuously adjust their hedge ratio as the underlying moves, converting the path-dependent option payout into a deterministic replication portfolio. The sports bettor cannot hedge: once a bet is placed, the position cannot be dynamically adjusted in response to game conditions (inning-by-inning, drive-by-drive). This means that the Black-Scholes risk-neutral pricing argument does not apply — there is no self-financing replication strategy.

**Discrete underlying.** The underlying "process" in sports betting is not a continuous diffusion but a sequence of discrete events (at-bats, innings, drives). Line movement between events is therefore not well-described by continuous Itô dynamics; a more accurate model would be a compound Poisson process with jumps at each event resolution.

**Jump processes from injury news.** The most significant departures from the continuous diffusion model arise from sudden jumps in probability driven by injury news. A starting pitcher scratched 90 minutes before game time causes an immediate, large jump in the implied probability — more consistent with a **jump-diffusion model** (Merton 1976) than with continuous GBM. In the options pricing language, injury events are analogous to earnings announcements: sudden jumps that GBM cannot capture.

**No risk-neutral measure.** The Black-Scholes framework requires the existence of a risk-neutral measure under which discounted asset prices are martingales. This is guaranteed in complete markets with no arbitrage. Sports betting markets are decidedly incomplete: outcomes cannot be replicated by trading in other instruments, and no portfolio of bets can create a risk-free position (beyond trivial arbitrage across books). The absence of a risk-neutral measure means that risk-neutral pricing formulas cannot be applied to sports bets without additional assumptions about risk preferences.

**Practical value of the analogy despite limitations.** Despite these limitations, the Black-Scholes analogy provides genuine value. The conceptual mapping of line movement to price dynamics, of line dispersion to implied volatility, and of bet timing to optimal stopping gives practitioners a powerful vocabulary for reasoning about sports betting strategy. The uncertainty-adjusted Kelly fraction derived from this framework is practically useful and has empirical support. The key is to treat the Black-Scholes framework as a source of **intuition and approximate tools**, not as an exact pricing engine for sports bets.

---

## Key Takeaways

A moneyline bet is structurally equivalent to a binary cash-or-nothing option, with the stake as premium and the win probability as the risk-neutral probability $\Phi(d_2)$. This structural equivalence motivates applying the rich machinery of options pricing theory to sports betting. Line movement between bet open and game time can be modeled as a stochastic process — specifically a log-odds Brownian motion or mean-reverting Ornstein-Uhlenbeck process — providing a principled basis for reasoning about bet timing.

Under a zero-drift (martingale) model of the closing line, the expected CLV from waiting is zero, and bets should be placed as soon as positive EV is identified. Under mean-reversion, a non-trivial optimal stopping boundary exists that balances the expected improvement in odds against the variance of line movement. The American option optimal stopping framework provides the formal solution.

Model uncertainty in the probability estimate $\hat{p}$ reduces the optimal Kelly fraction by a multiplicative factor $1 - \sigma_p^2 / (\hat{p}(1-\hat{p}))$, where $\sigma_p$ is the standard deviation of estimation error. Cross-book line dispersion provides an observable proxy for $\sigma_p$, enabling a dispersion-adjusted Kelly fraction that reduces position sizes on high-uncertainty games.

The Black-Scholes analogy has genuine limitations: no continuous hedging is available, the discrete event nature of sports creates jump dynamics not captured by GBM, injury news creates sudden jumps analogous to earnings announcements, and the absence of a complete market means no formal risk-neutral measure exists. Despite these limitations, the analogy provides valuable conceptual and quantitative tools that have direct practical applications in bet sizing and timing.
