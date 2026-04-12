---
layout: post
title: "Fundamental Theory: Probability, Edge, and the Mathematics of Sports Betting"
date: 2026-04-10
categories: research
tags: [sports-betting, quantitative-finance, kelly-criterion]
mathjax: true
excerpt: "A rigorous derivation of the mathematical foundations underlying sports betting markets, from moneyline odds to expected value and the concept of edge."
---

## Introduction

Sports betting markets are, at their core, markets in probability. Every transaction between a bettor and a bookmaker is an agreement on the likelihood of a binary outcome, denominated in cash flows contingent on that outcome. Understanding the mathematical structure of these transactions — how odds encode probability, how bookmakers extract margin, and what it means to hold a structural edge — is the prerequisite for any serious quantitative approach to this domain.

This article derives these foundations from first principles. The treatment assumes graduate-level familiarity with probability theory and is intended for practitioners who are comfortable with expectation operators, measure-theoretic probability, and basic optimization. No results are stated without derivation.

---

## 1. The Moneyline Bet as a Binary Contingent Claim

Consider a sporting contest between two teams, Home and Away. Let $\Omega = \{H, A\}$ be the sample space, where $H$ denotes a Home win and $A$ an Away win. (For simplicity we treat the contest as a binary outcome; draws and extra-innings extensions are handled by market conventions not central to this discussion.) Define a probability measure $\mathbb{P}$ on $\Omega$ such that $\mathbb{P}(H) = p$ and $\mathbb{P}(A) = 1 - p$ for some true probability $p \in (0, 1)$.

A **moneyline bet** on the Home team is a contingent claim paying a fixed net profit $b > 0$ per unit staked if the event $H$ occurs, and losing the unit stake $c = 1$ if $A$ occurs. The payoff function $\pi: \Omega \to \mathbb{R}$ is therefore

$$
\pi(\omega) =
\begin{cases}
b & \text{if } \omega = H \\
-c & \text{if } \omega = A
\end{cases}
\tag{1}
$$

This is precisely the structure of a **binary cash-or-nothing derivative**: the holder pays a premium $c$ upfront and receives a fixed payout $b + c$ if the underlying event occurs. The parallel with financial derivatives is not merely aesthetic; the same pricing principles apply, and we will exploit this correspondence explicitly in Article 6.

### 1.1 American Odds and Implied Probability

In US markets, moneyline odds are quoted in American format. There are two cases: the **favorite** (negative American odds) and the **underdog** (positive American odds). Let $M$ denote the American moneyline quote.

**Case 1: Favorite ($M < 0$).** The convention is that $|M|$ units must be risked to win 100 units of net profit. Thus for a unit stake:

$$
b_{\text{fav}} = \frac{100}{|M|} = \frac{-100}{M}
\tag{2}
$$

since $M < 0$. The breakeven condition — the probability at which the bet has zero expected value — is found by setting $\mathbb{E}[\pi] = 0$:

$$
p_{\text{implied}} \cdot b_{\text{fav}} - (1 - p_{\text{implied}}) \cdot c = 0
\tag{3}
$$

Substituting $c = 1$ and solving for $p_{\text{implied}}$:

$$
\begin{align}
p_{\text{implied}} \cdot b_{\text{fav}} &= 1 - p_{\text{implied}} \tag{4} \\
p_{\text{implied}} (b_{\text{fav}} + 1) &= 1 \tag{5} \\
p_{\text{implied}} &= \frac{1}{b_{\text{fav}} + 1} \tag{6}
\end{align}
$$

Substituting $b_{\text{fav}} = -100 / M$:

$$
p_{\text{implied}} = \frac{1}{\frac{-100}{M} + 1} = \frac{1}{\frac{-100 + M}{M}} = \frac{M}{M - 100}
\tag{7}
$$

For $M < 0$, both numerator and denominator are negative, so $p_{\text{implied}} > 0$. For example, with $M = -150$:

$$
p_{\text{implied}} = \frac{-150}{-150 - 100} = \frac{-150}{-250} = 0.60
\tag{8}
$$

This says that at $-150$, the bookmaker's implied probability is 60%.

**Case 2: Underdog ($M > 0$).** The convention is that 100 units risked wins $M$ units of net profit. For a unit stake:

$$
b_{\text{dog}} = \frac{M}{100}
\tag{9}
$$

The same breakeven derivation gives:

$$
p_{\text{implied}} = \frac{1}{b_{\text{dog}} + 1} = \frac{1}{\frac{M}{100} + 1} = \frac{100}{100 + M}
\tag{10}
$$

For example, with $M = +130$:

$$
p_{\text{implied}} = \frac{100}{100 + 130} = \frac{100}{230} \approx 0.4348
\tag{11}
$$

**Summary.** Defining the net decimal odds $b$ (profit per unit staked) in each case via equations (2) and (9), the implied probability is uniformly given by:

$$
p_{\text{implied}} = \frac{1}{b + 1} = \frac{1}{d}
\tag{12}
$$

where $d = b + 1$ is the gross decimal odds (total return per unit staked including stake recovery).

---

## 2. The Vigorish and the Bookmaker's Overround

A rational bookmaker does not simply price both sides at the true probability — this would leave no margin. Instead, the bookmaker inflates the implied probabilities on both sides of the market, ensuring that the sum exceeds unity. This inflation is called the **vigorish** (or vig), and its aggregate measure is the **overround**.

### 2.1 Deriving the Overround

Let $p_H^{\text{implied}}$ and $p_A^{\text{implied}}$ denote the implied probabilities extracted from the bookmaker's Home and Away moneyline quotes via equation (12). Define the **overround** $\rho$ as:

$$
\rho = p_H^{\text{implied}} + p_A^{\text{implied}} - 1
\tag{13}
$$

**Claim:** $\rho > 0$ guarantees the bookmaker a positive expected margin.

**Proof.** Consider a bettor who, having observed the overround, bets both sides in proportion to their implied probabilities. Let the bettor stake $s_H$ on Home and $s_A$ on Away. The bookmaker takes in $s_H + s_A$ in total stakes. If Home wins, the bookmaker pays out $s_H \cdot d_H = s_H / p_H^{\text{implied}}$. If Away wins, the bookmaker pays out $s_A \cdot d_A = s_A / p_A^{\text{implied}}$.

For the bookmaker to retain margin regardless of outcome, we require the payout in each scenario to be less than total stakes. Setting $s_H = p_H^{\text{implied}}$ and $s_A = p_A^{\text{implied}}$ (proportional staking):

- Total stakes: $s_H + s_A = p_H^{\text{implied}} + p_A^{\text{implied}} = 1 + \rho$
- Payout if Home wins: $s_H \cdot (1 / p_H^{\text{implied}}) = 1$
- Payout if Away wins: $s_A \cdot (1 / p_A^{\text{implied}}) = 1$

In both scenarios, the bookmaker collects $1 + \rho$ and pays out $1$, retaining $\rho > 0$. Since $\rho > 0$ iff $p_H^{\text{implied}} + p_A^{\text{implied}} > 1$, the bookmaker's margin is positive if and only if the overround is positive. $\square$

### 2.2 Numerical Example

Consider a game where the bookmaker posts $-115$ on both Home and Away. The implied probabilities are:

$$
p_H^{\text{implied}} = \frac{115}{115 + 100} = \frac{115}{215} \approx 0.5349
\tag{14}
$$

$$
p_A^{\text{implied}} = \frac{115}{215} \approx 0.5349
\tag{15}
$$

$$
\rho = 0.5349 + 0.5349 - 1 = 0.0698
\tag{16}
$$

The bookmaker's overround is approximately 7.0%. This means that on each dollar of action, the book retains approximately $0.70 in expectation — a substantial margin that must be overcome by any positive-expectation strategy.

---

## 3. Vig Removal: Recovering Fair Probabilities

To assess whether a bet offers positive expected value, we need an estimate of the true probability. While this is ultimately a modeling problem (addressed in Article 3), we can first extract the market's best estimate by removing the vig from the bookmaker's lines.

### 3.1 Multiplicative Normalization

The **fair probability** (vig-removed implied probability) for the Home side is defined as:

$$
p_H^{\text{fair}} = \frac{p_H^{\text{implied}}}{p_H^{\text{implied}} + p_A^{\text{implied}}}
\tag{17}
$$

and symmetrically for Away:

$$
p_A^{\text{fair}} = \frac{p_A^{\text{implied}}}{p_H^{\text{implied}} + p_A^{\text{implied}}}
\tag{18}
$$

By construction, $p_H^{\text{fair}} + p_A^{\text{fair}} = 1$, so this defines a valid probability measure on $\{H, A\}$.

### 3.2 Uniqueness of Multiplicative Normalization

**Claim:** Among all linear normalizations of the form $p_i^{\text{fair}} = \alpha_i \cdot p_i^{\text{implied}} + \beta_i$ that produce a valid probability measure, multiplicative normalization ($\beta_i = 0$) is the unique one that preserves the odds ratio.

**Proof.** The odds ratio between Home and Away is:

$$
\text{OR} = \frac{p_H^{\text{implied}} / (1 - p_H^{\text{implied}})}{p_A^{\text{implied}} / (1 - p_A^{\text{implied}})}
\tag{19}
$$

We require the fair probabilities to satisfy $p_H^{\text{fair}} / p_A^{\text{fair}} = p_H^{\text{implied}} / p_A^{\text{implied}}$ (odds ratio preservation). Under multiplicative normalization:

$$
\frac{p_H^{\text{fair}}}{p_A^{\text{fair}}} = \frac{p_H^{\text{implied}} / (p_H^{\text{implied}} + p_A^{\text{implied}})}{p_A^{\text{implied}} / (p_H^{\text{implied}} + p_A^{\text{implied}})} = \frac{p_H^{\text{implied}}}{p_A^{\text{implied}}}
\tag{20}
$$

This confirms odds ratio preservation. For any affine normalization $p_i^{\text{fair}} = \alpha p_i^{\text{implied}} + \beta$ with $\beta \neq 0$:

$$
\frac{p_H^{\text{fair}}}{p_A^{\text{fair}}} = \frac{\alpha p_H^{\text{implied}} + \beta}{\alpha p_A^{\text{implied}} + \beta} \neq \frac{p_H^{\text{implied}}}{p_A^{\text{implied}}}
\tag{21}
$$

unless $p_H^{\text{implied}} = p_A^{\text{implied}}$, which is not generally the case. Therefore multiplicative normalization is the unique odds-ratio-preserving linear normalization. $\square$

### 3.3 Example

Returning to the $-115 / -115$ example: since both sides have identical implied probabilities, the fair probabilities are trivially $p_H^{\text{fair}} = p_A^{\text{fair}} = 0.50$, as expected for a pick-em game priced with symmetric vig.

Consider now a game with $M_H = -150$ and $M_A = +130$:

$$
p_H^{\text{implied}} = \frac{150}{250} = 0.600, \quad p_A^{\text{implied}} = \frac{100}{230} \approx 0.4348
\tag{22}
$$

$$
p_H^{\text{implied}} + p_A^{\text{implied}} \approx 1.0348, \quad \rho \approx 0.0348
\tag{23}
$$

$$
p_H^{\text{fair}} = \frac{0.600}{1.0348} \approx 0.5799, \quad p_A^{\text{fair}} \approx \frac{0.4348}{1.0348} \approx 0.4201
\tag{24}
$$

The market's vig-free estimate of the Home win probability is approximately 58.0%.

---

## 4. Expected Value: Full Derivation from First Principles

Let $p \in (0,1)$ denote the bettor's estimate of the true win probability for a given side, and let $b > 0$ be the net odds (profit per unit staked). The bettor stakes $c > 0$ units. Define the net payoff random variable $\Pi$:

$$
\Pi =
\begin{cases}
b \cdot c & \text{with probability } p \\
-c & \text{with probability } 1 - p
\end{cases}
\tag{25}
$$

The **expected value** of the bet is:

$$
\mathbb{E}[\Pi] = p \cdot (b \cdot c) + (1-p) \cdot (-c) = c \left[ p \cdot b - (1-p) \right]
\tag{26}
$$

For a unit stake ($c = 1$):

$$
EV = p \cdot b - (1 - p)
\tag{27}
$$

### 4.1 The Positive EV Condition

The bet has positive expected value if and only if $EV > 0$:

$$
\begin{align}
p \cdot b - (1-p) &> 0 \tag{28} \\
p \cdot b &> 1 - p \tag{29} \\
p \cdot b + p &> 1 \tag{30} \\
p(b + 1) &> 1 \tag{31} \\
p &> \frac{1}{b+1} \tag{32}
\end{align}
$$

But from equation (12), $p_{\text{implied}} = 1/(b+1)$. Therefore:

$$
EV > 0 \iff p > p_{\text{implied}}
\tag{33}
$$

The bet has positive expected value if and only if the bettor's true probability estimate exceeds the bookmaker's implied probability. This is the central condition for a positive-expectation bet.

### 4.2 Numerical Example

Consider a Home team priced at $M_H = -130$, giving $b = 100/130 \approx 0.7692$ and $p_{\text{implied}} = 130/230 \approx 0.5652$. Suppose our model estimates $p = 0.60$. Then:

$$
EV = 0.60 \times 0.7692 - (1 - 0.60) = 0.4615 - 0.40 = 0.0615
\tag{34}
$$

Per unit staked, this bet returns an expected $0.0615$ — a 6.15% edge per dollar risked.

---

## 5. The Edge Metric

### 5.1 Definition

The **edge** of a bet is defined as the difference between the bettor's true probability estimate and the vig-removed market implied probability:

$$
\text{edge} = p_{\text{true}} - p_{\text{fair}}
\tag{35}
$$

where $p_{\text{fair}}$ is computed via equation (17). Note carefully that edge is measured against the *fair* implied probability, not the raw implied probability — this ensures the edge metric is not inflated by the amount of vig in the market.

### 5.2 Relationship Between Edge and EV

We now derive the exact relationship between edge as defined in equation (35) and expected value.

Let $p = p_{\text{true}}$ and let the market fair probability be $p_{\text{fair}}$. The net odds $b$ are derived from the bookmaker's raw (vigged) implied probability $p_{\text{implied}}$:

$$
b = \frac{1}{p_{\text{implied}}} - 1 = \frac{1 - p_{\text{implied}}}{p_{\text{implied}}}
\tag{36}
$$

The expected value (from equation 27) is:

$$
EV = p \cdot b - (1-p) = p \cdot \frac{1-p_{\text{implied}}}{p_{\text{implied}}} - (1-p)
\tag{37}
$$

$$
= \frac{p(1 - p_{\text{implied}}) - p_{\text{implied}}(1-p)}{p_{\text{implied}}}
\tag{38}
$$

$$
= \frac{p - p \cdot p_{\text{implied}} - p_{\text{implied}} + p \cdot p_{\text{implied}}}{p_{\text{implied}}}
\tag{39}
$$

$$
= \frac{p - p_{\text{implied}}}{p_{\text{implied}}}
\tag{40}
$$

Now write $p_{\text{implied}} = p_{\text{fair}} \cdot (1 + \rho)$ (since $p_{\text{fair}} = p_{\text{implied}} / (1 + \rho)$ from normalization, where $1 + \rho$ is the total overround sum). Thus:

$$
EV = \frac{p - p_{\text{fair}}(1+\rho)}{p_{\text{fair}}(1+\rho)} = \frac{1}{1+\rho} \cdot \frac{p - p_{\text{fair}}(1+\rho)}{p_{\text{fair}}}
\tag{41}
$$

$$
= \frac{1}{1+\rho} \left( \frac{p}{p_{\text{fair}}} - (1+\rho) \right) = \frac{p - p_{\text{fair}}}{p_{\text{fair}}(1+\rho)} - 1 + \frac{1}{1+\rho}
\tag{42}
$$

It is cleaner to express this exactly as:

$$
EV = \frac{p - p_{\text{implied}}}{p_{\text{implied}}}
\tag{43}
$$

For the **approximate relationship** when $\rho$ is small (which is the standard approximation used in practice), write $p_{\text{implied}} \approx p_{\text{fair}}$:

$$
EV \approx \frac{p - p_{\text{fair}}}{p_{\text{fair}}} = \frac{\text{edge}}{p_{\text{fair}}}
\tag{44}
$$

Since $p_{\text{fair}} < 1$, the expected value per unit staked exceeds the edge in absolute terms, by a factor of $1/p_{\text{fair}}$. For a near-even-money bet where $p_{\text{fair}} \approx 0.5$, the expected value is approximately twice the edge.

**Exact relationship.** Substituting the exact expression:

$$
EV = \frac{\text{edge} + p_{\text{true}} - p_{\text{true}}}{p_{\text{implied}}} = \frac{p_{\text{true}} - p_{\text{implied}}}{p_{\text{implied}}}
\tag{45}
$$

With $p_{\text{implied}} = p_{\text{fair}} \cdot (p_H^{\text{implied}} + p_A^{\text{implied}})$, the exact form in terms of edge is:

$$
EV = \frac{\text{edge}}{p_{\text{fair}} \cdot (1 + \rho)} - \frac{\rho}{1 + \rho}
\tag{46}
$$

This exact formula shows that the vig acts as a drag on EV: even a bet with positive edge can have reduced expected value if the overround $\rho$ is large.

### 5.3 Numerical Example

Continuing the previous example: $M_H = -130$, $p_{\text{implied}} \approx 0.5652$. Suppose the two-sided overround is $\rho = 0.04$ (4%). Then:

$$
p_{\text{fair}} = \frac{0.5652}{1.04} \approx 0.5435
\tag{47}
$$

With $p_{\text{true}} = 0.60$:

$$
\text{edge} = 0.60 - 0.5435 = 0.0565
\tag{48}
$$

$$
EV = \frac{0.0565}{0.5435 \times 1.04} \approx \frac{0.0565}{0.5652} \approx 0.100 - \frac{0.04}{1.04} \approx 0.0615
\tag{49}
$$

This agrees with the direct calculation in equation (34), confirming the relationship.

---

## 6. Why Positive EV Is Necessary But Not Sufficient

### 6.1 The Role of Variance

A positive expected value guarantees profitability only in the limit of infinitely many bets. In any finite sequence, the realized outcome can diverge substantially from the expectation due to variance. The variance of a single bet (unit stake) is:

$$
\text{Var}[\Pi] = \mathbb{E}[\Pi^2] - (\mathbb{E}[\Pi])^2 = p \cdot b^2 + (1-p) \cdot 1 - EV^2
\tag{50}
$$

For a moneyline bet at $-110$ (i.e., $b \approx 0.909$) with $p = 0.55$:

$$
\text{Var}[\Pi] = 0.55 \times 0.826 + 0.45 \times 1 - (0.55 \times 0.909 - 0.45)^2
\tag{51}
$$

$$
= 0.454 + 0.450 - (0.050)^2 = 0.904 - 0.0025 \approx 0.90
\tag{52}
$$

The standard deviation per bet is approximately $\sqrt{0.90} \approx 0.95$ units — nearly equal to the stake. By contrast, the expected profit is only $0.05$ units. The signal-to-noise ratio at the level of a single bet is roughly $0.05 / 0.95 \approx 5\%$.

### 6.2 The Risk of Ruin

Even with positive EV, betting a fixed dollar amount each time introduces the **risk of ruin**: the probability that cumulative losses exhaust the bankroll before the positive expectation manifests. For a symmetric random walk with absorbing barriers, the ruin probability is:

$$
\mathbb{P}(\text{ruin}) = \frac{(q/p)^N - (q/p)^k}{(q/p)^N - 1}
\tag{53}
$$

where $p$ is the per-bet win probability, $q = 1 - p$, $k$ is current wealth in units, and $N$ is the target wealth. Even for $p = 0.52$ (positive edge), starting with $k = 100$ units, the ruin probability is non-negligible for modest bankrolls.

This analysis motivates the need for optimal position sizing. A bet with positive EV can still ruin a bettor who stakes too aggressively. The Kelly Criterion — derived in detail in Article 2 — provides the optimal solution to this position sizing problem by maximizing the long-run geometric growth rate of the bankroll, implicitly controlling ruin probability. The key insight is that position sizing is not merely a risk management afterthought: it is the mechanism by which a theoretical edge in expected value is converted into realized long-run wealth.

---

## Key Takeaways

A moneyline bet is a binary contingent claim whose payoff structure maps directly to a cash-or-nothing binary option. The conversion between American odds and implied probability follows from the zero-EV breakeven condition and yields the formula $p_{\text{implied}} = 1 / (b+1)$ where $b$ is the net decimal odds. Bookmakers embed a positive overround $\rho > 0$ into their markets by inflating both sides' implied probabilities above unity in sum; this guarantees the book a positive margin on every dollar of action.

The fair (vig-removed) probability is recovered via multiplicative normalization, which is the unique linear normalization that preserves the odds ratio. A bet has positive expected value if and only if the bettor's true probability estimate exceeds the bookmaker's implied probability — the condition $p > p_{\text{implied}}$ is both necessary and sufficient for $EV > 0$. The edge metric $p_{\text{true}} - p_{\text{fair}}$ captures the bettor's structural advantage over the vig-removed market consensus, and the expected value per unit staked is approximately edge divided by fair probability for near-even-money markets.

Positive expected value is, however, necessary but not sufficient for long-run profitability. The high variance inherent in binary outcomes relative to the magnitude of typical edges creates substantial short-run noise, and naive flat-betting introduces a meaningful risk of ruin even for genuinely positive-EV bettors. These considerations make optimal position sizing — the subject of Article 2 — not merely a risk management tool but a mathematical prerequisite for translating edge into compound wealth growth.
