---
layout: post
title: "Real World Frictions: Why Theory and Practice Diverge in Sports Betting Markets"
date: 2026-04-10
categories: research
tags: [sports-betting, quantitative-finance, kelly-criterion]
mathjax: true
excerpt: "A rigorous treatment of the practical frictions that limit real-world sports betting returns: account restrictions, market impact, closing line bias, regulatory constraints, bankroll scaling, and the sample size problem."
---

## Introduction

The Kelly Criterion and Bayesian probability estimation provide the theoretical foundation for a positive-expectation sports betting strategy. In practice, however, the gap between theoretical edge and realized returns is substantial and predictable. Market frictions — account restrictions, market impact, benchmark bias, regulatory fragmentation, bankroll constraints, and the statistical challenge of validating small edges — systematically reduce what a quantitative bettor can extract from a model with genuine predictive power.

This article catalogs these frictions mathematically. The goal is not to discourage quantitative approaches but to ensure that return expectations are calibrated to the realistic operating environment rather than to theoretical limits. Each friction is modeled explicitly, and where possible, we derive the reduction in effective expected returns.

---

## 1. The Account Restriction Problem

### 1.1 The Lifecycle of a Winning Account

Regulated US sportsbooks operate under a business model in which recreational bettors (who lose in aggregate) subsidize promotional spending and platform economics. Winners — particularly systematic winners whose patterns resemble sharp rather than recreational betting — disrupt this model and are subject to bet limits, stake restrictions, or account closure.

Let $\tau$ denote the time (in days) before a winning account is restricted to minimum bet sizes, rendering it effectively unusable for systematic betting. Empirically, $\tau$ is a decreasing function of bet size and win rate. We model the expected account lifespan as:

$$
\mathbb{E}[\tau] = \frac{C}{f \cdot W_0 \cdot \text{edge}}
\tag{1}
$$

where $C$ is a book-specific constant (capturing how aggressively the book monitors accounts), $f$ is the Kelly fraction (proportional to edge), $W_0$ is the starting bankroll, and $\text{edge}$ is the systematic edge per bet. The product $f \cdot W_0 \cdot \text{edge}$ is proportional to the expected profit per bet — the signal visible to the book's risk management system.

This is a rough model, but the key insight is clear: the higher the per-bet profit, the shorter the expected account life. A bettor placing 2% edge bets at $\$200$ per game triggers restriction sooner than one placing 0.5% edge bets at $\$50$ per game, even if both extract the same long-run EV per dollar of bankroll.

### 1.2 Impact on Effective Kelly Fraction

Let $\bar{f}$ denote the maximum fraction per bet above which the account is detected and restricted within the current season. The effective Kelly fraction is:

$$
f_{\text{eff}} = \min(f^*, \bar{f})
\tag{2}
$$

If $f^* > \bar{f}$, the bettor is forced to underbet relative to the Kelly optimum. From the fractional Kelly growth rate analysis in Article 2, the growth rate under $f_{\text{eff}} = \alpha f^*$ (with $\alpha = \bar{f}/f^*$) is:

$$
G(f_{\text{eff}}) \approx \frac{\mu^2}{\sigma^2}\left(\alpha - \frac{\alpha^2}{2}\right)
\tag{3}
$$

For $\alpha = 0.5$ (restriction limits to half-Kelly), the effective growth rate is 75% of theoretical maximum. For $\alpha = 0.1$ (severe restriction), the growth rate drops to $0.1 - 0.005 = 0.095$ times the theoretical maximum — roughly 10% of what the edge would otherwise support.

### 1.3 Mitigation Strategies

Account fragmentation across multiple sportsbooks is the primary mitigation, as it allows the full Kelly fraction to be spread across books without any single book observing the full bet size. The effective Kelly fraction per book becomes $f_{\text{eff}} / B$ for $B$ books, delaying restriction onset. We address the regulatory and practical constraints on multi-book operations in Section 4.

---

## 2. Market Impact and Line Movement

### 2.1 The Optimal Execution Analog

In financial markets, a large order moves the price against the trader, reducing the profit from the trade. The analogous effect in sports betting is line movement: a large bet causes the book's offered probability to shift toward the bet side, reducing the edge available for subsequent bets on the same game.

Let $L_0$ denote the fair probability at the time of betting, and let $S$ denote the stake. We model the post-bet fair probability as:

$$
L(S) = L_0 + \lambda S
\tag{4}
$$

where $\lambda > 0$ is the **market impact coefficient** (units: probability per dollar). This is a linear (constant market impact) model, analogous to the simplest model used in optimal execution theory.

### 2.2 Deriving Optimal Bet Size

The expected value of a bet of size $S$ at entry probability $L_0$ against a true probability $p$ is:

$$
EV(S) = (p - L_0) \cdot S \cdot \frac{1}{L_0}
\tag{5}
$$

Wait — we must be more careful. The market impact means that as the bet is placed, the odds adjust. If we model the execution as instantaneous at price $L(S) = L_0 + \lambda S$ (i.e., the final price is the post-impact price), then the net edge at execution is:

$$
\text{edge}(S) = p - L(S) = p - L_0 - \lambda S = \text{edge}_0 - \lambda S
\tag{6}
$$

where $\text{edge}_0 = p - L_0 > 0$ is the pre-impact edge. The expected profit from a bet of size $S$ with post-impact edge is:

$$
\pi(S) = \text{edge}(S) \cdot S \cdot \frac{1}{L(S)} \approx (\text{edge}_0 - \lambda S) \cdot S \cdot \frac{1}{L_0}
\tag{7}
$$

where we approximate $L(S) \approx L_0$ for small $\lambda S$. Maximizing with respect to $S$:

$$
\frac{d\pi}{dS} = \frac{1}{L_0}(\text{edge}_0 - 2\lambda S) = 0
\tag{8}
$$

$$
S^* = \frac{\text{edge}_0}{2\lambda}
\tag{9}
$$

The optimal bet size is half of the bet that would eliminate all edge. This is exactly the result from optimal execution theory: with linear market impact, the profit-maximizing quantity is half the quantity at which impact equals edge. The maximum expected profit is:

$$
\pi(S^*) = \frac{\text{edge}_0}{2\lambda} \cdot \frac{\text{edge}_0}{2L_0} = \frac{\text{edge}_0^2}{4\lambda L_0}
\tag{10}
$$

**Numerical example.** Suppose $\text{edge}_0 = 0.03$ (3% edge on a fair probability of $L_0 = 0.52$), and the market impact coefficient is $\lambda = 0.001$ per $\$100$ staked (i.e., betting $\$100$ moves the line by 0.1 percentage points). Then:

$$
S^* = \frac{0.03}{2 \times 0.001 / 100} = \frac{0.03}{0.00002} = 1500
\tag{11}
$$

The profit-maximizing bet is $\$1,500$, yielding expected profit:

$$
\pi(S^*) = \frac{(0.03)^2}{4 \times (0.001/100) \times 0.52} = \frac{0.0009}{0.0000208} \approx \$43.27
\tag{12}
$$

For comparison, the unconstrained Kelly fraction on a $\$10,000$ bankroll at 3% edge and $L_0 = 0.52$ would suggest a bet of approximately $f^* \times W_0 \approx 0.062 \times 10,000 = \$620$, well below the market-impact optimal of $\$1,500$. In this regime, the Kelly constraint binds before the market impact constraint.

---

## 3. The Closing Line Benchmark Problem

### 3.1 Bias in CLV Estimates from Noisy Closing Lines

**Closing Line Value (CLV)** is defined and analyzed extensively in Article 5. A key practical limitation is that the closing line used as benchmark is itself a noisy estimate of true probability. Let $L_T^*$ denote the true probability at game time and $L_T$ denote the observed closing line (vig-removed). If the closing line is unbiased:

$$
L_T = L_T^* + \epsilon, \quad \mathbb{E}[\epsilon] = 0, \quad \text{Var}(\epsilon) = \sigma_C^2
\tag{13}
$$

The observed CLV for a bet placed at $L_t$ is:

$$
\widehat{CLV} = L_T - L_t = (L_T^* - L_t) + \epsilon = CLV^* + \epsilon
\tag{14}
$$

The observed CLV is an unbiased estimator of true CLV when the closing line is unbiased. However, the **variance** of the CLV estimator is increased by the noise in the closing line:

$$
\text{Var}(\widehat{CLV}) = \text{Var}(CLV^*) + \sigma_C^2
\tag{15}
$$

This additional variance reduces the statistical efficiency of CLV as an edge signal. Smaller books with lower liquidity tend to have noisier closing lines ($\sigma_C^2$ is larger), inflating the variance of the CLV estimate and requiring larger samples to achieve the same statistical power.

### 3.2 Implications of Using Recreational Book Consensus

A further complication arises when the closing line benchmark is derived from a consensus of recreational (square) books rather than sharp books. Recreational book closing lines tend to reflect public betting action more than sharp money, and may exhibit the **favorite-longshot bias**: favorites are systematically underpriced (i.e., their implied probability is too low) and underdogs are overpriced.

If the benchmark consistently underestimates the probability of favorites, then a bettor who systematically bets favorites will show artificially inflated CLV against this benchmark — positive CLV not because their entry timing is good, but because the benchmark is biased downward. Conversely, a bettor who bets underdogs will show depressed CLV against the same benchmark.

We address this by using a sharp-book-weighted consensus (Pinnacle, circa equivalent exchanges) as the primary CLV benchmark, and treating recreational book CLV as a secondary signal subject to explicit bias adjustment.

---

## 4. Regulatory Constraints by Jurisdiction

### 4.1 US State-Regulated Sportsbooks

As of early 2026, legal, state-regulated sports betting is available in approximately 35 US states. The major operators — DraftKings, FanDuel, BetMGM, Caesars — operate under state gaming commission licenses that require them to maintain customer protection standards and pay state tax rates ranging from 10% to 51% of gross gaming revenue.

These operators manage their liability via account restrictions (as modeled in Section 1) and maximum bet limits. For the quantitative bettor, the state-regulated market offers legal clarity and consumer protection but limited capacity (typical per-game limits of $\$500$–$\$5,000$ for restricted accounts) and aggressive winner restriction.

### 4.2 CFTC-Regulated Prediction Markets

Kalshi and related CFTC-regulated prediction market operators represent an alternative pathway. These platforms are regulated as designated contract markets (DCMs) under the Commodity Exchange Act, not as gaming operators. The regulatory structure provides several advantages: no state-by-state licensing patchwork, more transparent market microstructure, and in principle, greater capacity for large positions.

The disadvantage is reduced liquidity relative to the major sportsbooks, and pricing that may lag the sharp sportsbook market due to lower participation from professional bettors. For jurisdictions without legal sportsbooks, prediction markets provide a fully legal alternative with comparable pricing efficiency in major events.

### 4.3 Offshore Operators

Offshore operators (those operating without US state licenses) offer in some cases better lines, fewer restrictions, and higher limits. However, they operate in a legal gray area for US bettors and carry substantial counterparty risk: account balances are not protected by any US regulatory framework, and disputes have no recourse through US courts. We do not recommend or model offshore operations in this platform.

---

## 5. Bankroll Scaling Constraints

### 5.1 Return as a Function of Bankroll and Edge

Let $W_0$ be the starting bankroll, $N$ be the number of bets per day, $f^*$ be the average Kelly fraction, and $\text{edge}$ be the average edge per bet. The expected daily profit is approximately:

$$
\text{Profit}_{\text{day}} \approx N \cdot f^* \cdot W_0 \cdot b \cdot \text{edge} = N \cdot EV \cdot W_0
\tag{16}
$$

where we use $f^* = EV/b$ from Article 2, giving $f^* b = EV$.

Annualizing over 180 MLB betting days:

$$
\text{Profit}_{\text{annual}} \approx 180 \cdot N \cdot EV \cdot W_0
\tag{17}
$$

### 5.2 Minimum Bankroll for Meaningful Returns

We define "meaningful absolute returns" as generating at least $\$10,000$ per year from the strategy. Rearranging equation (17):

$$
W_0 \geq \frac{\$10,000}{180 \cdot N \cdot EV}
\tag{18}
$$

**Case 1: Edge = 0.5%, $N = 5$ bets per day, fair odds ($L_0 = 0.5$, $b = 1$).** $EV = \text{edge} \cdot (b+1)/1 \approx 2 \times 0.005 = 0.010$ per bet (using the approximation from Article 1).

$$
W_0 \geq \frac{10,000}{180 \times 5 \times 0.010} = \frac{10,000}{9} \approx \$1,111
\tag{19}
$$

This suggests a surprisingly low minimum bankroll — but this ignores that at 0.5% edge the Kelly fraction is very small ($f^* \approx 0.005$ for fair odds), so absolute bet sizes are tiny. For $W_0 = \$1,000$, each bet is approximately $\$5$ — below minimum bet limits at most books. The binding constraint is not the bankroll formula but the minimum bet size.

**Case 2: Edge = 2%, $N = 5$ bets per day.** $EV \approx 0.040$ per bet.

$$
W_0 \geq \frac{10,000}{180 \times 5 \times 0.040} = \frac{10,000}{36} \approx \$278
\tag{20}
$$

Again, the formula suggests a trivially small bankroll, but the binding constraint is minimum bet limits and account restriction risk at higher absolute bet sizes. The practical minimum bankroll for $\$10,000$ annual returns at 2% edge without triggering restriction is approximately $\$10,000$–$\$25,000$, which supports bet sizes of $\$200$–$\$500$ per game — below the typical restriction threshold for a new account.

---

## 6. The Sample Size Problem

### 6.1 Statistical Power for Edge Detection

Let $\delta > 0$ denote the true edge per bet (in probability units), and let $\alpha_{\text{sig}} \in (0,1)$ be the significance level and $\beta_{\text{pow}} \in (0,1)$ be the type-II error rate (so $1 - \beta_{\text{pow}}$ is the statistical power). We want to determine the minimum number of bets $N_{\min}$ required to detect edge $\delta$ with significance $\alpha_{\text{sig}}$ and power $1 - \beta_{\text{pow}}$.

**Setup.** Under the null hypothesis $H_0: \delta = 0$, the sample mean outcome $\bar{X}$ (with $X_i = 1$ if bet $i$ wins, $0$ if it loses) has standard deviation:

$$
\sigma_X = \sqrt{\frac{p_0(1-p_0)}{N}}
\tag{21}
$$

where $p_0 = p_{\text{implied}}$ is the implied win probability under the null. The test statistic is:

$$
Z = \frac{\bar{X} - p_0}{\sigma_X} = \frac{(\bar{X} - p_0) \sqrt{N}}{\sqrt{p_0(1-p_0)}}
\tag{22}
$$

Under the alternative $H_1: \delta = p_{\text{true}} - p_0 > 0$, the expected value of $Z$ is:

$$
\mathbb{E}[Z | H_1] = \frac{\delta \sqrt{N}}{\sqrt{p_0(1-p_0)}}
\tag{23}
$$

The power condition requires:

$$
\frac{\delta \sqrt{N}}{\sqrt{p_0(1-p_0)}} \geq z_{\alpha_{\text{sig}}} + z_{\beta_{\text{pow}}}
\tag{24}
$$

where $z_q = \Phi^{-1}(1-q)$ is the standard normal quantile. Solving for $N$:

$$
N_{\min} = \frac{(z_{\alpha_{\text{sig}}} + z_{\beta_{\text{pow}}})^2 \cdot p_0(1-p_0)}{\delta^2}
\tag{25}
$$

### 6.2 Numerical Results

**Case 1: $\delta = 0.02$ (2% edge), $\alpha_{\text{sig}} = 0.05$, power = 80%.** With $p_0 = 0.50$, $z_{0.05} = 1.645$, $z_{0.20} = 0.842$:

$$
N_{\min} = \frac{(1.645 + 0.842)^2 \times 0.25}{(0.02)^2} = \frac{6.184 \times 0.25}{0.0004} = \frac{1.546}{0.0004} \approx 3,865
\tag{26}
$$

**Case 2: $\delta = 0.005$ (0.5% edge), same significance and power:**

$$
N_{\min} = \frac{6.184 \times 0.25}{(0.005)^2} = \frac{1.546}{0.000025} \approx 61,840
\tag{27}
$$

These results explain why outcome-based validation requires extremely large samples: even a 2% edge requires approximately 4,000 bets, and a 0.5% edge requires over 60,000. At 5 bets per day over a 180-day MLB season, 4,000 bets takes approximately 4.4 seasons. Detection of smaller edges is statistically infeasible via outcome data alone.

### 6.3 CLV-Based Validation: The 100-200 Bet Advantage

Closing Line Value (CLV) is a more statistically efficient edge metric because its variance is much lower than that of binary outcomes. Let $\sigma_{\text{CLV}}^2$ denote the variance of individual CLV observations. Empirically, $\sigma_{\text{CLV}} \approx 0.03$–$0.05$ (in probability units) for typical MLB games, compared to $\sigma_X \approx \sqrt{0.25} = 0.50$ for binary outcomes.

The sample size formula for CLV-based edge detection is:

$$
N_{\min}^{\text{CLV}} = \frac{(z_{\alpha_{\text{sig}}} + z_{\beta_{\text{pow}}})^2 \cdot \sigma_{\text{CLV}}^2}{\delta^2}
\tag{28}
$$

With $\sigma_{\text{CLV}} = 0.04$ and $\delta = 0.02$:

$$
N_{\min}^{\text{CLV}} = \frac{6.184 \times (0.04)^2}{(0.02)^2} = \frac{6.184 \times 0.0016}{0.0004} = \frac{0.00989}{0.0004} \approx 25
\tag{29}
$$

Wait — let us reconsider. The CLV metric measures the probability improvement between entry and close, not the binary outcome. Its variance is:

$$
\text{Var}(\text{CLV}_i) = \text{Var}(L_T - L_t)
\tag{30}
$$

This is the variance of line movement, which is substantially smaller than the variance of a binary $\{0,1\}$ outcome. The ratio of required sample sizes is:

$$
\frac{N_{\min}^{\text{outcome}}}{N_{\min}^{\text{CLV}}} = \frac{p_0(1-p_0)}{\sigma_{\text{CLV}}^2} \approx \frac{0.25}{(0.04)^2} = \frac{0.25}{0.0016} \approx 156
\tag{31}
$$

CLV-based validation requires approximately 150 times fewer observations than outcome-based validation to achieve the same statistical power. For a 2% edge, this reduces the required sample from approximately 4,000 bets to approximately 26 bets — or roughly one week of betting activity. Article 5 develops the full statistical theory of CLV versus P&L as edge metrics.

---

## Key Takeaways

Real-world sports betting returns fall substantially short of theoretical edge-based projections due to a cluster of identifiable frictions. Account restrictions compress the effective Kelly fraction to a fraction of the theoretical optimum, with the magnitude depending on per-bet profit visibility and book-specific restriction policies. Market impact reduces effective edge as a function of bet size, and the profit-maximizing bet size under linear impact is half the edge-eliminating quantity — an exact analog of the Almgren-Chriss framework from financial market microstructure.

Closing line benchmarks introduce additional noise into CLV estimates, with recreational book consensus exhibiting systematic biases that inflate CLV for favorite-side bettors. The regulatory landscape in the US bifurcates into state-licensed sportsbooks (higher liquidity, aggressive winner restriction), CFTC-regulated prediction markets (legal across jurisdictions, lower liquidity), and offshore operators (not recommended due to counterparty risk). Bankroll scaling is governed by expected annual return formulas, but the binding constraints in practice are minimum bet limits and restriction thresholds rather than the Kelly formula itself.

The sample size requirement for outcome-based edge validation is prohibitive: detecting a 2% edge at 80% power requires approximately 4,000 bets, and detecting a 0.5% edge requires over 60,000. CLV-based validation is approximately 150 times more statistically efficient, enabling edge detection within weeks rather than years. This distinction has major practical implications for model evaluation and is the subject of the dedicated analysis in Article 5.
