---
layout: post
title: "Ensemble Forecasting and the Brier Score: Measuring Combined Predictive Power"
date: 2026-04-13
categories: research
tags: [sports-betting, quantitative-finance, ensemble-methods, brier-score, forecasting]
mathjax: true
excerpt: "How combining independent probability sources reduces forecast variance, and why source disagreement identifies the highest-value betting opportunities."
---

## Introduction

A probability forecast is more than a number — it is a claim. The forecaster asserts that the true probability of some event equals *p*, and that claim can be right or wrong in ways that accumulate systematically over many observations. The central question of forecast evaluation is: given a sequence of probability forecasts and their associated outcomes, how do we measure how well the forecaster understood the underlying probability structure of the world?

The **Brier score**, introduced by Glenn W. Brier in 1950, is the canonical answer to this question. It is a proper scoring rule — a metric that rewards honest reporting of beliefs — and it admits a rich algebraic decomposition into bias, variance, and resolution components that illuminate exactly where a forecasting system succeeds or fails. In the context of sports betting, where we operate multiple independent probability sources (statistical models, public analytics platforms, historical Elo ratings), the Brier score provides both a diagnostic and a decision-making tool.

This article develops the mathematics of the Brier score from first principles, derives the theoretical guarantees of ensemble forecasting, connects source disagreement to a computable "diversity bonus," and traces the implications through to Kelly position sizing. The treatment assumes graduate-level probability theory and familiarity with the expectation operator.

---

## 1. The Brier Score as a Proper Scoring Rule

### 1.1 Definition

Let $\{(p_i, o_i)\}_{i=1}^{N}$ be a sequence of $N$ forecast-outcome pairs, where $p_i \in [0,1]$ is the predicted probability assigned to the event before it occurs, and $o_i \in \{0,1\}$ is the observed outcome ($o_i = 1$ if the event occurs, $o_i = 0$ otherwise). The **Brier score** is the mean squared error between forecasts and outcomes:

$$
BS = \frac{1}{N} \sum_{i=1}^{N} (p_i - o_i)^2
\tag{1}
$$

Lower values of $BS$ indicate better forecast accuracy. A perfect forecaster who always assigns $p_i = 1$ to events that occur and $p_i = 0$ to events that do not would achieve $BS = 0$. An uninformative forecaster who always predicts the base rate $\bar{o} = \frac{1}{N}\sum_{i=1}^N o_i$ achieves a reference score against which we measure skill.

### 1.2 Proper Scoring Rule Property

**Definition.** A scoring rule $S(p, o)$ is **proper** if, for any forecaster who believes the true probability of the event is $q$, the expected score is minimized (for negatively oriented rules, as Brier score is) when the forecaster reports $p = q$. A proper scoring rule eliminates any strategic incentive to misreport one's true beliefs.

**Theorem.** The Brier score is a proper scoring rule.

**Proof.** Consider a single forecast-outcome pair. The forecaster believes the true probability of $o = 1$ is $q \in (0,1)$, but reports $p \in [0,1]$. The expected Brier score for this single observation, over the randomness in the outcome $o \sim \text{Bernoulli}(q)$, is:

$$
\mathbb{E}_q[(p - o)^2] = q(p - 1)^2 + (1-q)(p - 0)^2
\tag{2}
$$

Expanding:

$$
\begin{align}
\mathbb{E}_q[(p - o)^2] &= q(p^2 - 2p + 1) + (1-q)p^2 \tag{3} \\
&= p^2 q - 2pq + q + p^2 - p^2 q \tag{4} \\
&= p^2 - 2pq + q \tag{5}
\end{align}
$$

To minimize over the reported probability $p$, we take the derivative and set it equal to zero:

$$
\frac{d}{dp}\left[p^2 - 2pq + q\right] = 2p - 2q = 0 \implies p = q
\tag{6}
$$

The second derivative $\frac{d^2}{dp^2}[p^2 - 2pq + q] = 2 > 0$, confirming this is a global minimum. Therefore the expected Brier score is minimized when the forecaster reports their true belief $p = q$. $\square$

### 1.3 The Cost of Misreporting

The theorem establishes that honest reporting is optimal. We can quantify the cost of deliberate misreporting. If the forecaster reports $p \neq q$, the expected Brier score exceeds the honest minimum by a computable penalty. From equation (5), the expected score at truthful reporting $p = q$ is:

$$
\mathbb{E}_q[(q - o)^2] = q^2 - 2q^2 + q = q - q^2 = q(1-q)
\tag{7}
$$

For any misreported $p$, the expected score is:

$$
\mathbb{E}_q[(p - o)^2] = p^2 - 2pq + q
\tag{8}
$$

The excess cost of misreporting is therefore:

$$
\mathbb{E}_q[(p - o)^2] - \mathbb{E}_q[(q - o)^2] = (p^2 - 2pq + q) - (q - q^2) = p^2 - 2pq + q^2 = (p - q)^2
\tag{9}
$$

The penalty for misreporting is exactly $(p - q)^2$ — the squared distance between the reported and true probability. This is a clean, interpretable result: the expected Brier score increases quadratically in the degree of misreporting.

In sports betting applications, this property is critical. A modeler who distorts their probability estimate — say, by reporting 0.60 when they believe 0.55, hoping to appear more confident — will see their Brier score deteriorate by $(0.60 - 0.55)^2 = 0.0025$ per game on average, a substantial drag that compounds over a season. The Brier score thus functions as a disciplining mechanism that rewards epistemic honesty.

---

## 2. The Bias-Variance-Resolution Decomposition

### 2.1 Murphy (1973) Decomposition

The overall Brier score can be decomposed into three interpretable components that diagnose different failure modes of a forecasting system. We follow the decomposition of Murphy (1973).

**Notation.** Let $\bar{p} = \frac{1}{N}\sum_{i=1}^N p_i$ denote the mean forecast, $\bar{o} = \frac{1}{N}\sum_{i=1}^N o_i$ the empirical base rate, and define sample variance and covariance in the standard way:

$$
\mathbb{V}[p] = \frac{1}{N}\sum_{i=1}^N (p_i - \bar{p})^2, \quad \mathbb{V}[o] = \frac{1}{N}\sum_{i=1}^N (o_i - \bar{o})^2, \quad \text{Cov}(p, o) = \frac{1}{N}\sum_{i=1}^N (p_i - \bar{p})(o_i - \bar{o})
\tag{10}
$$

**Theorem (Murphy Decomposition).** The Brier score admits the decomposition:

$$
BS = (\bar{p} - \bar{o})^2 + \mathbb{V}[p] - 2\,\text{Cov}(p, o) + \mathbb{V}[o]
\tag{11}
$$

**Proof.** We expand the squared error directly:

$$
\begin{align}
BS &= \frac{1}{N}\sum_{i=1}^N (p_i - o_i)^2 \tag{12} \\
&= \frac{1}{N}\sum_{i=1}^N \left[(p_i - \bar{p}) - (o_i - \bar{o}) + (\bar{p} - \bar{o})\right]^2 \tag{13}
\end{align}
$$

Let $\tilde{p}_i = p_i - \bar{p}$, $\tilde{o}_i = o_i - \bar{o}$, and $\delta = \bar{p} - \bar{o}$, so the summand is $(\tilde{p}_i - \tilde{o}_i + \delta)^2$. Expanding:

$$
(\tilde{p}_i - \tilde{o}_i + \delta)^2 = \tilde{p}_i^2 + \tilde{o}_i^2 + \delta^2 - 2\tilde{p}_i\tilde{o}_i + 2\delta\tilde{p}_i - 2\delta\tilde{o}_i
\tag{14}
$$

Summing over $i$ and dividing by $N$, and using the fact that $\frac{1}{N}\sum_i \tilde{p}_i = \frac{1}{N}\sum_i \tilde{o}_i = 0$ (by construction of the centered quantities):

$$
BS = \mathbb{V}[p] + \mathbb{V}[o] + \delta^2 - 2\,\text{Cov}(p,o)
\tag{15}
$$

Substituting $\delta = \bar{p} - \bar{o}$ gives equation (11). $\square$

### 2.2 Interpretation of Each Component

The four terms in equation (11) have natural interpretations:

- $(\bar{p} - \bar{o})^2$ is the **bias-squared** term. It measures the average tendency of the forecaster to overestimate or underestimate. A well-calibrated forecaster has $\bar{p} \approx \bar{o}$, making this term small.

- $\mathbb{V}[p]$ is the **variance** of the forecasts. High forecast variance means the forecaster is willing to make bold predictions far from the mean; low variance means predictions cluster near $\bar{p}$. This term penalizes spread in forecasts.

- $2\,\text{Cov}(p, o)$ is the **resolution** component. High covariance means forecasts move in the same direction as outcomes — when the forecaster predicts high probabilities, events actually occur more often. This reduces the Brier score and is the component we want to maximize.

- $\mathbb{V}[o] = \bar{o}(1 - \bar{o})$ (for binary outcomes) is the **climatological variance**, determined entirely by the base rate of events. This is fixed regardless of the forecasting system.

### 2.3 Variance Reduction via Ensembling

The key insight is that **ensembling reduces $\mathbb{V}[p]$ without affecting the resolution term** (assuming the ensemble mean forecast has the same covariance with outcomes as the individual sources). Since $\mathbb{V}[p]$ enters the Brier score with a positive sign, reducing forecast variance directly improves (lowers) $BS$.

**Numerical Example.** Suppose a single forecasting source has $\mathbb{V}[p] = 0.020$, $(\bar{p} - \bar{o})^2 = 0.001$, $\text{Cov}(p,o) = 0.015$, and $\mathbb{V}[o] = 0.25$ (reflecting $\bar{o} = 0.5$).

The single-source Brier score is:

$$
BS_{\text{single}} = 0.001 + 0.020 - 2(0.015) + 0.25 = 0.241
\tag{16}
$$

Now ensemble three such independent sources with equal weights $w_k = 1/3$. As we will derive formally in Section 3, ensembling three independent sources with equal individual variance $\sigma^2 = 0.020$ reduces forecast variance to:

$$
\mathbb{V}[p_{\text{ensemble}}] = \frac{\sigma^2}{K} = \frac{0.020}{3} \approx 0.0067
\tag{17}
$$

Assuming the ensemble preserves the bias and covariance structure:

$$
BS_{\text{ensemble}} = 0.001 + 0.0067 - 2(0.015) + 0.25 = 0.2277
\tag{18}
$$

The ensemble reduces $BS$ by $0.241 - 0.228 = 0.013$, purely through variance compression. This is a non-trivial improvement achieved without any additional signal — just averaging.

---

## 3. Ensemble Variance Reduction

### 3.1 Setup and Assumptions

We now derive the variance reduction properties of ensemble forecasting formally.

**Assumptions:** (i) There are $K$ independent probability sources, indexed $k = 1, \ldots, K$. (ii) Each source has equal individual forecast variance $\sigma^2 = \mathbb{V}[p_k]$ for all $k$. (iii) Sources receive equal weights $w_k = 1/K$ in the ensemble.

The ensemble forecast is the simple average:

$$
p_{\text{ensemble}} = \frac{1}{K} \sum_{k=1}^{K} p_k
\tag{19}
$$

### 3.2 Variance Under Independence

**Theorem.** Under assumptions (i)–(iii) with mutually independent sources ($\text{Cov}(p_j, p_k) = 0$ for $j \neq k$), the ensemble forecast variance is:

$$
\mathbb{V}[p_{\text{ensemble}}] = \frac{\sigma^2}{K}
\tag{20}
$$

**Proof.** By the bilinearity of variance for independent random variables:

$$
\begin{align}
\mathbb{V}\!\left[\frac{1}{K}\sum_{k=1}^K p_k\right] &= \frac{1}{K^2} \mathbb{V}\!\left[\sum_{k=1}^K p_k\right] \tag{21} \\
&= \frac{1}{K^2} \sum_{k=1}^K \mathbb{V}[p_k] + \frac{1}{K^2}\sum_{j \neq k} \text{Cov}(p_j, p_k) \tag{22} \\
&= \frac{1}{K^2} \cdot K\sigma^2 + 0 \tag{23} \\
&= \frac{\sigma^2}{K} \tag{24}
\end{align}
$$

$\square$

This result says forecast variance shrinks by a factor of $1/K$ under independence. Going from one source to three sources under independence reduces variance to one-third of the original — a substantial compression.

### 3.3 Variance Under Correlated Sources

In practice, sources are not fully independent. FanGraphs, ESPN Analytics, and an Elo model all observe the same games, the same team histories, and are influenced by the same public information. We expect positive correlations between their probability estimates.

**Theorem.** Let $\rho \in [0,1)$ be the common pairwise correlation between any two sources, so that $\text{Cov}(p_j, p_k) = \rho \sigma^2$ for all $j \neq k$. Then:

$$
\mathbb{V}[p_{\text{ensemble}}] = \sigma^2 \left(\frac{1}{K} + \frac{K-1}{K}\rho\right)
\tag{25}
$$

**Proof.** Starting from equation (22) and substituting $\text{Cov}(p_j, p_k) = \rho\sigma^2$:

$$
\begin{align}
\mathbb{V}[p_{\text{ensemble}}] &= \frac{1}{K^2}\left[K\sigma^2 + K(K-1)\rho\sigma^2\right] \tag{26} \\
&= \frac{\sigma^2}{K^2}\left[K + K(K-1)\rho\right] \tag{27} \\
&= \frac{\sigma^2}{K}\left[1 + (K-1)\rho\right] \tag{28} \\
&= \sigma^2\left(\frac{1}{K} + \frac{K-1}{K}\rho\right) \tag{29}
\end{align}
$$

$\square$

**Corollary.** For any $\rho < 1$ and $K \geq 2$, the ensemble variance is strictly less than the individual variance:

$$
\mathbb{V}[p_{\text{ensemble}}] = \sigma^2\left(\frac{1}{K} + \frac{K-1}{K}\rho\right) < \sigma^2
\tag{30}
$$

This follows because $\frac{1}{K} + \frac{K-1}{K}\rho < 1$ whenever $\rho < 1$, which we can verify:

$$
\frac{1}{K} + \frac{K-1}{K}\rho < 1 \iff \frac{K-1}{K}\rho < \frac{K-1}{K} \iff \rho < 1
\tag{31}
$$

Ensembling is *always* beneficial (in terms of variance reduction) provided the sources are not perfectly correlated.

### 3.4 Quantifying Diminishing Returns

**Numerical Example.** For $K = 3$ independent sources ($\rho = 0$), variance reduction factor is $1/3 \approx 0.333$. Under the realistic correlation $\rho = 0.3$ between our three sources:

$$
\text{Variance reduction factor} = \frac{1}{3} + \frac{2}{3}(0.3) = 0.333 + 0.200 = 0.533
\tag{32}
$$

Under independence, variance would drop to 33.3% of the original. Under $\rho = 0.3$, it drops only to 53.3% — the inter-source correlation absorbs roughly half the theoretical benefit. Still, a 47% variance reduction is substantial and translates directly to improved Brier scores as shown in Section 2.3.

The key practical lesson is that adding a fourth or fifth source with high correlation to existing sources yields diminishing returns. The marginal benefit of source $K+1$ over source $K$ in ensemble variance reduction is:

$$
\Delta \mathbb{V} = \mathbb{V}_K - \mathbb{V}_{K+1} = \sigma^2 \cdot \frac{(1-\rho)}{K(K+1)}
\tag{33}
$$

This diminishes as $K^{-2}$, meaning the jump from one to two sources is far more valuable than the jump from ten to eleven.

---

## 4. The Brier Skill Score

### 4.1 Definition

The **Brier Skill Score** (BSS) measures a forecasting system's improvement over a reference baseline, expressed as a proportion of the maximum possible improvement. Using the market consensus probability as the reference:

$$
BSS = 1 - \frac{BS_{\text{model}}}{BS_{\text{reference}}}
\tag{34}
$$

where $BS_{\text{reference}}$ is the Brier score achieved by the reference forecaster (here, the market-implied probability). A $BSS > 0$ indicates the model beats the market; $BSS = 0$ indicates parity; $BSS < 0$ indicates the model is worse than the market.

### 4.2 Empirical Results for Our Three Sources

Using the market consensus baseline $BS_{\text{reference}} = 0.2578$ (computed over $n = 162$ games), we compute BSS for each source:

For **FanGraphs** ($n = 152$ games, $BS_{\text{FG}} = 0.2234$):

$$
BSS_{\text{FG}} = 1 - \frac{0.2234}{0.2578} = 1 - 0.8665 = 0.133
\tag{35}
$$

For **ESPN Analytics** ($n = 200$ games, $BS_{\text{ESPN}} = 0.2380$):

$$
BSS_{\text{ESPN}} = 1 - \frac{0.2380}{0.2578} = 1 - 0.9232 = 0.077
\tag{36}
$$

For **Elo Model** ($n = 66$ games, $BS_{\text{Elo}} = 0.2481$):

$$
BSS_{\text{Elo}} = 1 - \frac{0.2481}{0.2578} = 1 - 0.9624 = 0.038
\tag{37}
$$

All three sources beat the market, with FanGraphs leading at $BSS = 0.133$, followed by ESPN at $BSS = 0.077$ and Elo at $BSS = 0.038$.

### 4.3 Ensemble BSS Under Equal Weighting

Under equal weights $w_k = 1/K$, the ensemble Brier score approximates the mean of individual source Brier scores (to first order, ignoring covariance terms between sources and outcomes):

$$
BSS_{\text{ensemble}} \approx 1 - \frac{1}{K}\sum_{k=1}^K (1 - BSS_k) = \frac{1}{K}\sum_{k=1}^K BSS_k
\tag{38}
$$

**Assumptions for this approximation:** (i) Source Brier scores are computed over comparable games. (ii) The ensemble mean's covariance with outcomes approximately equals the mean of individual covariances.

Under equal weighting with our three sources:

$$
BSS_{\text{equal}} = \frac{0.133 + 0.077 + 0.038}{3} = \frac{0.248}{3} = 0.083
\tag{39}
$$

This equal-weighted ensemble achieves $BSS = 0.083$ — **worse than FanGraphs alone** ($BSS = 0.133$). Including lower-skill sources at equal weight dilutes the ensemble and drags performance toward the mean. This is the central tension of ensemble design: the variance reduction benefit must be weighed against the skill dilution cost.

### 4.4 Optimal Source Weighting

The resolution is skill-proportional weighting. Define the optimal weight for source $k$ as proportional to its skill, measured by the margin by which it beats the reference:

$$
w_k^* = \frac{BSS_k}{\sum_{j=1}^K BSS_j}
\tag{40}
$$

This allocates weight in proportion to demonstrated forecasting ability above the baseline. Weighting should reflect the *incremental value* each source adds, not just whether it beats the reference.

For our three sources, the total skill is $\sum BSS_k = 0.133 + 0.077 + 0.038 = 0.248$. The optimal weights are:

$$
w_{\text{FG}}^* = \frac{0.133}{0.248} = 0.536, \quad w_{\text{ESPN}}^* = \frac{0.077}{0.248} = 0.310, \quad w_{\text{Elo}}^* = \frac{0.038}{0.248} = 0.153
\tag{41}
$$

The optimally-weighted ensemble Brier score is:

$$
BS_{\text{optimal}} = \sum_{k} w_k^* \cdot BS_k = 0.536(0.2234) + 0.310(0.2380) + 0.153(0.2481)
\tag{42}
$$

Computing term by term:

$$
\begin{align}
BS_{\text{optimal}} &= 0.1197 + 0.0738 + 0.0380 \tag{43} \\
&= 0.2315 \tag{44}
\end{align}
$$

The corresponding skill score:

$$
BSS_{\text{optimal}} = 1 - \frac{0.2315}{0.2578} = 0.102
\tag{45}
$$

The skill-weighted ensemble at $BSS = 0.102$ outperforms the equal-weighted ensemble at $BSS = 0.083$ but still falls short of FanGraphs alone at $BSS = 0.133$. This pattern — the best individual source outperforming even the optimally-weighted ensemble — is common when one source significantly dominates and sources share substantial correlation. In such cases, the variance reduction benefit from ensembling must be balanced against the explicit advantage of upweighting the best source.

The platform's daily weight update mechanism implements this logic: source weights are set proportional to $\max(0, BS_{\text{market}} - BS_{\text{source}})$, which is monotone in the source's demonstrated skill advantage over the market. This operationalizes equation (40) in a rolling, data-driven way that adapts to changes in source quality over time.

---

## 5. The Ambiguity Decomposition

### 5.1 Deriving the Diversity Bonus

One of the most practically important results in ensemble forecasting theory is the **ambiguity decomposition**, which makes precise the intuition that "disagreement among sources improves the ensemble." We now derive this formally.

**Definition.** For a given game $i$ with source forecasts $\{p_{k,i}\}_{k=1}^K$ and ensemble forecast $p_{\text{ensemble},i} = \frac{1}{K}\sum_k p_{k,i}$, define the **ambiguity** as the within-game variance across source forecasts:

$$
A_i = \frac{1}{K}\sum_{k=1}^K (p_{k,i} - p_{\text{ensemble},i})^2
\tag{46}
$$

**Theorem (Ambiguity Decomposition).** The ensemble Brier score satisfies:

$$
BS_{\text{ensemble}} = \overline{BS}_{\text{sources}} - \bar{A}
\tag{47}
$$

where $\overline{BS}_{\text{sources}} = \frac{1}{K}\sum_k BS_k$ is the mean source Brier score and $\bar{A} = \frac{1}{N}\sum_{i=1}^N A_i$ is the mean ambiguity.

**Proof.** For a single game $i$, the ensemble squared error is:

$$
(p_{\text{ensemble},i} - o_i)^2 = \left(\frac{1}{K}\sum_k p_{k,i} - o_i\right)^2
\tag{48}
$$

The mean source squared error for game $i$ is:

$$
\frac{1}{K}\sum_k (p_{k,i} - o_i)^2
\tag{49}
$$

We will show that (49) minus (48) equals the ambiguity $A_i$. Expand:

$$
\begin{align}
\frac{1}{K}\sum_k (p_{k,i} - o_i)^2 &= \frac{1}{K}\sum_k \left[(p_{k,i} - p_{\text{ensemble},i}) + (p_{\text{ensemble},i} - o_i)\right]^2 \tag{50} \\
&= \frac{1}{K}\sum_k (p_{k,i} - p_{\text{ensemble},i})^2 + 2(p_{\text{ensemble},i} - o_i)\frac{1}{K}\sum_k(p_{k,i} - p_{\text{ensemble},i}) \\
&\quad + (p_{\text{ensemble},i} - o_i)^2 \tag{51}
\end{align}
$$

The middle term vanishes because $\frac{1}{K}\sum_k(p_{k,i} - p_{\text{ensemble},i}) = p_{\text{ensemble},i} - p_{\text{ensemble},i} = 0$ by definition of the ensemble mean. Therefore:

$$
\frac{1}{K}\sum_k (p_{k,i} - o_i)^2 = A_i + (p_{\text{ensemble},i} - o_i)^2
\tag{52}
$$

Rearranging:

$$
(p_{\text{ensemble},i} - o_i)^2 = \frac{1}{K}\sum_k (p_{k,i} - o_i)^2 - A_i
\tag{53}
$$

Averaging over all $N$ games:

$$
BS_{\text{ensemble}} = \overline{BS}_{\text{sources}} - \bar{A}
\tag{54}
$$

$\square$

### 5.2 Properties of Ambiguity

Several immediate consequences follow from equation (47).

**Nonnegativity.** Since $A_i = \frac{1}{K}\sum_k(p_{k,i} - p_{\text{ensemble},i})^2$ is a sum of squared deviations, $A_i \geq 0$ for all $i$, with $\bar{A} \geq 0$.

**Zero Ambiguity.** $A_i = 0$ if and only if $p_{k,i} = p_{\text{ensemble},i}$ for all $k$ — that is, all sources agree exactly. In this case, $BS_{\text{ensemble}} = \overline{BS}_{\text{sources}}$: the ensemble is no better than the average source.

**Diversity Bonus.** Whenever sources disagree ($A_i > 0$), the ensemble outperforms the average source: $BS_{\text{ensemble}} < \overline{BS}_{\text{sources}}$. The reduction is exactly equal to the mean ambiguity. This is the "diversity bonus" of ensemble forecasting — source disagreement is not a sign of trouble but a signal of value.

### 5.3 Numerical Illustration

**Game 1: Sources Agree (Low Ambiguity)**

Source forecasts: $p_{\text{FG}} = 0.55$, $p_{\text{ESPN}} = 0.54$, $p_{\text{Elo}} = 0.56$.

$$
p_{\text{ensemble}} = \frac{0.55 + 0.54 + 0.56}{3} = 0.55
\tag{55}
$$

$$
A_1 = \frac{(0.55-0.55)^2 + (0.54-0.55)^2 + (0.56-0.55)^2}{3} = \frac{0 + 0.0001 + 0.0001}{3} \approx 0.000067
\tag{56}
$$

The ensemble BS improvement over average sources is negligible: $\bar{A}_1 \approx 0.000067$.

**Game 2: Sources Disagree (High Ambiguity)**

Source forecasts: $p_{\text{FG}} = 0.62$, $p_{\text{ESPN}} = 0.51$, $p_{\text{Elo}} = 0.48$.

$$
p_{\text{ensemble}} = \frac{0.62 + 0.51 + 0.48}{3} = 0.537
\tag{57}
$$

$$
A_2 = \frac{(0.62-0.537)^2 + (0.51-0.537)^2 + (0.48-0.537)^2}{3} = \frac{0.006889 + 0.000729 + 0.003249}{3} \approx 0.00363
\tag{58}
$$

The ensemble BS improvement over average sources is $0.00363$ — more than **fifty times larger** than in Game 1.

The operational implication is striking: not all games benefit equally from ensembling. Games where sources agree offer minimal ensemble advantage — the ensemble merely reproduces what any individual source would have said. Games where sources disagree sharply are precisely the games where the ensemble adds the most value, compressing the error by canceling idiosyncratic deviations among sources.

---

## 6. Application to the Platform

### 6.1 Current Empirical State

The platform currently operates three active probability sources:

- **FanGraphs** ($n = 152$ games): $BS = 0.2234$, $BSS = 0.133$
- **ESPN Analytics** ($n = 200$ games): $BS = 0.2380$, $BSS = 0.077$
- **Elo Model** ($n = 66$ games): $BS = 0.2481$, $BSS = 0.038$
- **Market baseline** ($n = 162$ games): $BS = 0.2578$

All three sources beat the market consensus on their respective samples. FanGraphs leads substantially, with ESPN a meaningful second. The Elo model's $BSS = 0.038$ is positive but modest, reflecting the known limitations of purely ratings-based approaches that do not account for lineup changes, pitching rotations, or other game-specific information.

### 6.2 Daily Weight Update Mechanism

Source weights are updated each morning using a formula that directly operationalizes skill-proportional weighting:

$$
w_s \propto \max(0,\ BS_{\text{market}} - BS_s)
\tag{59}
$$

The market weight is assigned the residual after source weights are normalized, subject to a floor at 0.50:

$$
w_{\text{market}} = \max\!\left(0.50,\ 1 - \sum_s w_s\right)
\tag{60}
$$

The 50% floor on the market weight reflects an explicit Bayesian prior. With $n \approx 150$–$200$ games per source, confidence intervals on BSS are wide. A source with $BSS = 0.133$ over 152 games has a standard error of roughly:

$$
\text{SE}(BSS) \approx \frac{1}{\sqrt{N}} \cdot \frac{\sigma_{BS}}{BS_{\text{ref}}} \approx \frac{1}{\sqrt{152}} \cdot \frac{0.05}{0.2578} \approx 0.016
\tag{61}
$$

The signal is real but uncertain enough that maintaining substantial weight on the market — the aggregated wisdom of all public information — is the appropriate conservative response. As $N$ grows through the season, the floor can be relaxed and source weights can accumulate more aggressively.

### 6.3 Operationalizing Ambiguity as a Signal

The ambiguity decomposition suggests a direct operational use: compute the standard deviation of source forecasts for each game as a measure of **source dispersion**:

$$
\text{Dispersion}_i = \text{sd}(p_{\text{FG},i},\ p_{\text{ESPN},i},\ p_{\text{Elo},i})
\tag{62}
$$

High dispersion games are candidates for increased attention on two grounds. First, by the ambiguity decomposition, they are precisely the games where the ensemble adds the most value over individual sources — the ensemble posterior is most likely to be meaningfully different from and better than any individual source. Second, high dispersion suggests the market has not fully incorporated the information that at least one source is exploiting, creating a potential pricing inefficiency.

When the ensemble posterior disagrees with the market price and source dispersion is high, the bet hypothesis is at its strongest: multiple independent confirmation channels are pointing away from the market, and the ensemble's aggregation advantage is at its peak. Conversely, when dispersion is low and all sources agree with each other (but disagree with the market), the case rests entirely on a systematic blind spot in all sources simultaneously — a higher-risk scenario.

---

## 7. Connection to Kelly Sizing

### 7.1 Uncertainty-Adjusted Kelly Under Ensemble Variance

Article 6 derived an adjustment to the Kelly criterion for the case where the true probability *p* is not known but estimated with uncertainty. Suppose the true probability for a given game is:

$$
p = \hat{p} + \varepsilon, \quad \varepsilon \sim \mathcal{N}(0,\ \sigma_p^2)
\tag{63}
$$

where $\hat{p}$ is the ensemble forecast and $\sigma_p^2$ is the variance of the ensemble forecast (i.e., our uncertainty about *p*). The uncertainty-adjusted Kelly fraction derived in Article 6 is:

$$
f_{\text{adjusted}} = f^* \cdot \left(1 - \frac{\sigma_p^2}{p(1-p)}\right)
\tag{64}
$$

where $f^* = \frac{bp - (1-p)}{b}$ is the classical Kelly fraction for decimal-odds bet with net profit $b$ per unit staked.

### 7.2 Linking Ambiguity to Ensemble Variance

We now connect $\sigma_p^2$ — the uncertainty in the ensemble estimate — to the ambiguity $A_i$ from the decomposition in Section 5.

**Under the assumption of independent sources**, the variance of the ensemble mean as an estimator of the true probability is:

$$
\sigma_p^2 = \mathbb{V}[p_{\text{ensemble}}] = \frac{\sigma_{\text{source}}^2}{K}
\tag{65}
$$

The within-game source variance (ambiguity) is an estimate of $\sigma_{\text{source}}^2$ for that specific game:

$$
A_i \approx \sigma_{\text{source},i}^2
\tag{66}
$$

Therefore, the ensemble variance for game $i$ is approximated by:

$$
\sigma_{p,i}^2 \approx \frac{A_i}{K}
\tag{67}
$$

### 7.3 Ambiguity-Adjusted Kelly

Substituting equation (67) into equation (64), the **ambiguity-adjusted Kelly fraction** for game $i$ is:

$$
f_{\text{adjusted},i} = f_i^* \cdot \left(1 - \frac{A_i}{K \cdot \hat{p}_i(1 - \hat{p}_i)}\right)
\tag{68}
$$

This formula captures the intuition precisely: high ambiguity $A_i$ reduces the Kelly fraction, reflecting genuine uncertainty about the true probability; low ambiguity (sources agree) leaves the Kelly fraction near $f_i^*$. The denominator $K \cdot \hat{p}_i(1-\hat{p}_i)$ normalizes for the number of sources and the base uncertainty inherent to events near 50-50 probability.

**Numerical Example.** Consider a game with $\hat{p} = 0.55$ and decimal odds implying $b = 0.909$ (equivalent to -110 American odds). The classical Kelly fraction is:

$$
f^* = \frac{b \cdot \hat{p} - (1-\hat{p})}{b} = \frac{0.909(0.55) - 0.45}{0.909} = \frac{0.5000 - 0.45}{0.909} = \frac{0.0500}{0.909} = 0.055
\tag{69}
$$

(We use a half-Kelly of $f^* = 0.10$ for illustration, consistent with the platform's risk parameters.)

For the half-Kelly baseline $f^* = 0.10$ and $K = 3$ sources:

**Case 1: Low Ambiguity ($A_i = 0.0001$, sources agree as in Game 1 of Section 5.3):**

$$
f_{\text{adjusted}} = 0.10 \cdot \left(1 - \frac{0.0001}{3 \cdot 0.55 \cdot 0.45}\right) = 0.10 \cdot \left(1 - \frac{0.0001}{0.7425}\right) = 0.10 \cdot (1 - 0.000135) \approx 0.0999
\tag{70}
$$

The adjustment is negligible — sources agree, so we are confident in $\hat{p}$, and the position is near full half-Kelly.

**Case 2: High Ambiguity ($A_i = 0.004$, sources disagree as in Game 2 of Section 5.3):**

$$
f_{\text{adjusted}} = 0.10 \cdot \left(1 - \frac{0.004}{3 \cdot 0.55 \cdot 0.45}\right) = 0.10 \cdot \left(1 - \frac{0.004}{0.7425}\right) = 0.10 \cdot (1 - 0.00539) = 0.10 \cdot 0.9946 \approx 0.0995
\tag{71}
$$

Even at $A_i = 0.004$, the Kelly reduction under these parameters is small ($\approx 0.5\%$). However, for very high ambiguity games where, say, $A_i = 0.020$ (one source forecasts 0.70, another 0.45):

$$
f_{\text{adjusted}} = 0.10 \cdot \left(1 - \frac{0.020}{0.7425}\right) = 0.10 \cdot (1 - 0.0269) = 0.10 \cdot 0.9731 \approx 0.0973
\tag{72}
$$

A 2.7% reduction in the half-Kelly fraction. More consequentially, when bet sizing is done at full Kelly ($f^* = 0.20$ before the half-Kelly haircut), the impact doubles. And when $\hat{p}$ is near 0.55 with tight market odds — a common scenario for marginal edges — the relative size of the adjustment grows because the expected log-wealth gain is already small.

The practical takeaway is that the ambiguity signal should be monitored as a position-scaling variable: games where all sources sharply agree on a probability meaningfully different from the market price are the highest-confidence opportunities and warrant the largest positions. Games where sources sharply disagree deserve smaller positions even when the ensemble mean suggests positive edge, because the ensemble estimate itself carries genuine uncertainty.

---

## Key Takeaways

The Brier score is the correct metric for evaluating probability forecasters because it is a proper scoring rule — the only class of metrics that creates no strategic incentive to misreport true beliefs. The formal proof is elementary but consequential: the expected Brier score is minimized exactly when the reported probability equals the true probability, and misreporting by $\delta$ inflates the expected score by $\delta^2$. In a domain like sports betting where a modeler might be tempted to exaggerate confidence or shade estimates toward crowd-pleasing numbers, this mathematical structure creates a clean accountability mechanism. If we measure ourselves by Brier score, we are rewarded precisely for knowing what we know and nothing more.

Ensemble forecasting reduces forecast variance by a factor that depends on the number of sources and their pairwise correlations. Under full independence, three sources compress variance to one-third of the individual level; under realistic correlation $\rho = 0.3$, the compression is to 53% — still a meaningful improvement. This variance reduction maps directly to Brier score improvement through the Murphy decomposition, where forecast variance appears as an additive penalty. The variance reduction benefit is entirely free in the sense that it does not depend on any individual source being better than another; it depends only on sources not being perfectly correlated, a condition that holds as long as sources draw on different information or modeling philosophies.

The ambiguity decomposition — $BS_{\text{ensemble}} = \overline{BS}_{\text{sources}} - \bar{A}$ — provides a precise account of when ensemble averaging adds the most value. The ensemble always improves on the average source, with the improvement equal to the mean within-game variance across sources. This means source disagreement, often perceived as a sign of noise or confusion, is in fact the mechanism by which ensembles outperform. Games where FanGraphs, ESPN, and Elo give similar probabilities yield minimal ensemble benefit. Games where those sources spread across a wide range are precisely where aggregation cancels errors and the ensemble posterior is most trustworthy relative to any individual source.

Skill-proportional weighting outperforms equal weighting because including lower-skill sources at equal weight dilutes ensemble performance toward the mean of its components. The theoretical optimum allocates weight proportional to each source's demonstrated BSS — its margin over the market reference — and the platform's daily Brier-score weighting scheme approximates this optimum in a rolling, adaptive way. The 50% floor on market weight is not a hedge against model failure but a rational response to small-sample uncertainty: with 150-200 games per source, confidence intervals are wide enough that substantially discounting the market — the aggregated wisdom of thousands of participants — would be statistically unjustified. As sample sizes grow through the season, source weights can accumulate more aggressively.

Finally, ambiguity connects to Kelly position sizing through the uncertainty-adjusted Kelly framework: $f_{\text{adjusted}} = f^* \cdot (1 - A_i / (K \cdot \hat{p}(1-\hat{p})))$. High ambiguity means the ensemble forecast itself is uncertain, which should translate to a smaller bet even if the ensemble mean suggests positive expected value. This is not a contradiction but a logical extension of Kelly's foundational principle — bet in proportion to your true edge, and acknowledge that edge is uncertain when your best estimators disagree. The games where sources converge on a probability well away from the market price, with minimal inter-source dispersion, represent the strongest intersection of edge signal and estimation confidence, and deserve the largest positions.
