---
layout: post
title: "Technical Implementation: A Multi-Source Bayesian Probability Estimation Pipeline"
date: 2026-04-10
categories: research
tags: [sports-betting, quantitative-finance, kelly-criterion]
mathjax: true
excerpt: "A detailed walkthrough of the four-layer probability estimation pipeline, from multi-source data acquisition through Bayesian posterior blending and Elo-based modeling to market probability extraction."
---

## Introduction

A positive-expectation sports betting strategy requires two distinct capabilities: a model that estimates true win probabilities with sufficient accuracy to identify edges against the market, and an execution framework that translates those edges into optimal bet sizing. The mathematical theory of edge and sizing is developed in Articles 1 and 2. This article describes the implementation of the probability estimation component.

The central challenge is epistemic: no single data source provides a reliable, unbiased estimate of true win probabilities across all games and conditions. Analytical models based on historical statistics have predictive power but miss current injury information. Market prices aggregate diverse participant knowledge but contain vig and may reflect sharp money rather than true probability. Elo ratings capture long-run team quality but require adjustment for pitcher matchups. The solution is a Bayesian framework that combines these sources optimally, weighting each by its empirical track record.

---

## 1. System Architecture: The Four-Layer Pipeline

The platform is organized into four layers that transform raw data into executable bet recommendations.

**Layer 1: Data Acquisition.** Raw data is ingested from multiple APIs: ESPN Analytics for team-level statistics and real-time lineup data, FanGraphs for advanced pitching and hitting metrics, Odds API for current and historical moneyline prices across multiple sportsbooks, and a third-party Elo rating feed. Data freshness is critical: stale lineup or injury data can render probability estimates unreliable within hours of game time.

**Layer 2: Feature Engineering.** Raw statistics are transformed into probability-relevant features. Pitching features include the starting pitcher's Fielding Independent Pitching (FIP) metric, current-season ERA relative to league average, and handedness matchup adjustments. Team features include recent form (rolling win rates over 10, 20, and 40 games), home/away splits, rest days, and travel distance. All features are normalized to a common scale to prevent high-variance statistics from dominating the model.

**Layer 3: Strategy Optimization.** Probability estimates from Layer 2 are combined with market prices from multiple sportsbooks to identify positive-edge opportunities. Kelly fractions are computed and constrained as described in Article 2. The output is a ranked list of recommended bets with associated fractions, expected values, and confidence intervals.

**Layer 4: Evaluation.** Bet recommendations are logged against both actual outcomes and closing line values. This layer supports the diagnostic distinction between model accuracy (measured by CLV) and realized P&L, as developed in Article 5.

---

## 2. Probability Source Architecture

The pipeline incorporates three independent probability sources, each with distinct information content.

**Source 1: ESPN Analytics Model.** ESPN provides team win probability estimates derived from their proprietary analytics model. These estimates incorporate current-season statistics, home field advantage, and lineup information. The ESPN model is updated continuously as lineup cards are released, making it responsive to late-breaking information.

**Source 2: FanGraphs Projections.** FanGraphs publishes game-level win probability estimates using their ZiPS and Steamer projection systems. These projections weight historical performance and aging curves more heavily than current-season data, providing stability against small-sample noise. The FanGraphs estimates are particularly reliable for starting pitcher matchup quality.

**Source 3: Elo Rating Model.** Our internal Elo model tracks team quality ratings that are updated after each game. The Elo system provides a parsimonious baseline probability that is robust to the limitations of single-season statistics, particularly early in the season when sample sizes are small.

The three sources are treated as independent in the blending framework — an assumption we examine carefully below. To the extent that all three sources draw on the same underlying statistics (which they do, particularly with respect to publicly available team records), the independence assumption is an approximation. However, the sources differ substantially in how they weight current-season versus historical data and in how they handle pitching matchups, providing sufficient information diversity for the blending framework to add value.

---

## 3. The Bayesian Posterior Blending Framework

### 3.1 The Blended Posterior

Let $p_s^{(g)}$ denote the win probability estimate for game $g$ from source $s \in \{1, 2, 3\}$ (ESPN, FanGraphs, Elo), and let $p_{\text{market}}^{(g)}$ denote the vig-removed market probability. Define weights $w_s \geq 0$ for each model source and $w_{\text{market}} \geq 0$ for the market, with $\sum_s w_s + w_{\text{market}} = 1$. The posterior blended probability is:

$$
p_{\text{posterior}}^{(g)} = \sum_{s=1}^{3} w_s p_s^{(g)} + w_{\text{market}} p_{\text{market}}^{(g)}
\tag{1}
$$

This is an affine combination of probability estimates. The weights are estimated from historical data as described in Section 4.

### 3.2 Optimality Under Brier Score Minimization

**Claim:** Under the assumption that sources are mutually uncorrelated in their errors, equation (1) is the optimal linear combination of probability estimates in the sense of minimizing the expected Brier score.

**Proof.** The **Brier score** for a single game is $BS = (p_{\text{posterior}} - \mathbf{1}_{\text{win}})^2$, where $\mathbf{1}_{\text{win}} \in \{0, 1\}$ is the game outcome. The expected Brier score is:

$$
\mathbb{E}[BS] = \mathbb{E}[(p_{\text{posterior}} - \mathbf{1}_{\text{win}})^2]
\tag{2}
$$

Writing $p_{\text{posterior}} = \sum_k w_k p_k$ (where index $k$ ranges over all sources including market) and letting $p_{\text{true}}$ denote the true win probability:

$$
\mathbb{E}[BS] = \mathbb{E}\left[\left(\sum_k w_k p_k - \mathbf{1}_{\text{win}}\right)^2\right]
\tag{3}
$$

Adding and subtracting $p_{\text{true}}$:

$$
= \mathbb{E}\left[\left(\sum_k w_k p_k - p_{\text{true}} + p_{\text{true}} - \mathbf{1}_{\text{win}}\right)^2\right]
\tag{4}
$$

$$
= \mathbb{E}\left[\left(\sum_k w_k p_k - p_{\text{true}}\right)^2\right] + \text{Var}(\mathbf{1}_{\text{win}}) + \text{cross term}
\tag{5}
$$

The cross term vanishes when $\mathbb{E}[\mathbf{1}_{\text{win}} | p_k] = p_{\text{true}}$ (calibration condition). The irreducible variance term $\text{Var}(\mathbf{1}_{\text{win}}) = p_{\text{true}}(1 - p_{\text{true}})$ is independent of the weights. The minimization problem reduces to:

$$
\min_{\mathbf{w}: \sum w_k = 1} \mathbb{E}\left[\left(\sum_k w_k p_k - p_{\text{true}}\right)^2\right]
\tag{6}
$$

This is a constrained least-squares problem. Letting $\epsilon_k = p_k - p_{\text{true}}$ be the error of source $k$, and assuming $\mathbb{E}[\epsilon_k \epsilon_j] = 0$ for $k \neq j$ (uncorrelated errors):

$$
\mathbb{E}\left[\left(\sum_k w_k p_k - p_{\text{true}}\right)^2\right] = \mathbb{E}\left[\left(\sum_k w_k \epsilon_k\right)^2\right] = \sum_k w_k^2 \sigma_k^2
\tag{7}
$$

where $\sigma_k^2 = \mathbb{E}[\epsilon_k^2]$ is the mean squared error of source $k$. The optimal weights are:

$$
w_k^* = \frac{1/\sigma_k^2}{\sum_j 1/\sigma_j^2}
\tag{8}
$$

That is, the optimal weights are inversely proportional to each source's mean squared error. Sources with lower MSE (better calibration) receive higher weight. $\square$

**Assumptions for validity:** (i) Each source produces calibrated probabilities: $\mathbb{E}[p_k | p_{\text{true}}] = p_{\text{true}}$. (ii) Source errors are uncorrelated. (iii) The weighting scheme is linear. In practice, assumption (ii) is an approximation; positive error correlation would lead to overconfidence in the blended estimate, and we account for this through a conservative floor constraint on weights.

---

## 4. Empirical Weight Estimation via Brier Scores

### 4.1 The Brier Score

The **Brier Score** for source $k$ over a historical sample of $M$ games is:

$$
BS_k = \frac{1}{M} \sum_{g=1}^M (p_k^{(g)} - \mathbf{1}_{\text{win}}^{(g)})^2
\tag{9}
$$

Lower Brier scores indicate better calibration. The Brier score decomposes into calibration error, resolution, and uncertainty:

$$
BS_k = \underbrace{\frac{1}{M} \sum_g (p_k^{(g)} - \bar{p}_k)^2}_{\text{resolution}} - \underbrace{\frac{1}{M} \sum_g (\mathbf{1}_{\text{win}}^{(g)} - \bar{p})^2}_{\text{uncertainty}} + \underbrace{\frac{1}{M} \sum_g (\bar{p}_k - \bar{p})^2}_{\text{calibration}}
\tag{10}
$$

where $\bar{p} = M^{-1} \sum_g \mathbf{1}_{\text{win}}^{(g)}$ is the historical win rate and $\bar{p}_k = M^{-1} \sum_g p_k^{(g)}$ is the mean forecast. The uncertainty term is common to all sources and does not affect relative weighting.

### 4.2 Weight Computation with Floor Constraint

The empirical weight for source $k$ is computed as:

$$
\tilde{w}_k = \frac{1 / BS_k}{\sum_j 1 / BS_j}
\tag{11}
$$

A **floor constraint** $w_{\min} > 0$ is applied to prevent any source from receiving zero weight under small-sample conditions:

$$
w_k = \max(\tilde{w}_k, w_{\min})
\tag{12}
$$

followed by renormalization so $\sum_k w_k = 1$.

**Why the floor prevents weight collapse.** In finite samples, a source that has poor recent performance (high $BS_k$) due to sampling noise may receive near-zero weight, causing the blended estimate to ignore it entirely. If the source's true long-run accuracy is competitive, this sampling-driven exclusion degrades the blended estimate's calibration. The floor constraint ensures that all sources maintain a minimum contribution, providing insurance against temporary performance dips.

In our implementation we use $w_{\min} = 0.05$ for each of the three analytical sources and $w_{\min} = 0.15$ for the market probability, reflecting the market's generally superior calibration as a consensus of large-volume participants.

---

## 5. The Pitcher-Adjusted Elo Model

### 5.1 Standard Elo Update Equations

The Elo rating system assigns each team a scalar rating $R_i \in \mathbb{R}$. Before each game between teams $i$ and $j$, the expected win probability for team $i$ is:

$$
\mathbb{E}_{ij} = \frac{1}{1 + 10^{(R_j - R_i)/400}}
\tag{13}
$$

After the game, ratings are updated according to the actual outcome $S_{ij} \in \{1, 0\}$ (win/loss):

$$
R_i' = R_i + K \cdot (S_{ij} - \mathbb{E}_{ij})
\tag{14}
$$

$$
R_j' = R_j + K \cdot (S_{ji} - \mathbb{E}_{ji})
\tag{15}
$$

where $K$ is the update speed parameter. We use $K = 20$ for regular season games and $K = 24$ for playoff games, reflecting the greater information content of playoff performance.

The logistic form in equation (13) ensures that $\mathbb{E}_{ij} + \mathbb{E}_{ji} = 1$ for all rating pairs, making the system internally consistent as a probability measure.

### 5.2 Pitcher FIP Adjustment

The standard Elo system treats each game as reflecting team quality, but starting pitcher quality is a major determinant of game outcomes that is not captured in team-level ratings. We adjust the pre-game expected probability using the starting pitchers' FIP (Fielding Independent Pitching):

$$
\Delta_{\text{pitch}} = \theta \cdot (\overline{FIP}_{\text{league}} - FIP_{\text{home}}) - \theta \cdot (\overline{FIP}_{\text{league}} - FIP_{\text{away}})
\tag{16}
$$

where $\theta$ is a calibrated scaling parameter and $\overline{FIP}_{\text{league}}$ is the current-season league average FIP. The adjustment $\Delta_{\text{pitch}}$ is added to the Elo rating differential before computing expected probabilities:

$$
p_{\text{Elo,adj}}^{(g)} = \frac{1}{1 + 10^{(R_j - R_i + \Delta_{\text{pitch}})/400}}
\tag{17}
$$

Lower FIP (better pitcher performance) increases the home team's expected win probability by raising the effective rating differential. The parameter $\theta$ is calibrated by minimizing the Brier score of Elo predictions on historical hold-out data.

### 5.3 Regression to the Mean

At the start of each season, team ratings are regressed toward the league mean $\bar{R} = 1500$:

$$
R_i^{\text{new season}} = R_i^{\text{end of season}} + \delta \cdot (\bar{R} - R_i^{\text{end of season}}) = (1-\delta) R_i^{\text{end}} + \delta \bar{R}
\tag{18}
$$

We use $\delta = 0.33$ (33% regression). This reflects the empirical observation that team quality regresses substantially year-over-year due to roster turnover, coaching changes, and the inherent unpredictability of player development. A value of $\delta = 0$ would carry over last season's ratings unchanged; $\delta = 1$ would reset all teams to league average. The 33% figure is consistent with the literature on sports Elo systems and our own calibration exercises.

---

## 6. Market Probability Estimation via Vig Removal

### 6.1 Consensus Fair Probability

The market probability is computed from a consensus of multiple sportsbooks rather than a single book, to reduce dependence on any individual book's pricing errors or idiosyncrasies. Let $p_{\text{implied},k}^{H}$ and $p_{\text{implied},k}^{A}$ denote the Home and Away implied probabilities from book $k$ for $k = 1, \ldots, B$ books. The consensus fair probability is:

$$
p_{\text{market}}^{H} = \frac{1}{B} \sum_{k=1}^{B} \frac{p_{\text{implied},k}^{H}}{p_{\text{implied},k}^{H} + p_{\text{implied},k}^{A}}
\tag{19}
$$

Averaging the vig-removed probabilities (rather than averaging the raw odds and then removing vig) is preferred because the vig removal operation is nonlinear: $\text{vig\_remove}(\text{avg}(p)) \neq \text{avg}(\text{vig\_remove}(p))$ in general.

### 6.2 Cross-Book Line Dispersion as an Information Signal

The **dispersion** of fair probabilities across books is defined as:

$$
\sigma_{\text{books}}^{(g)} = \text{std}\left(\left\{p_{\text{fair},k}^{H,(g)}\right\}_{k=1}^B\right)
\tag{20}
$$

High dispersion indicates that books disagree significantly on the true probability, which often signals stale or conflicting information — for example, an injury rumor that some books have already priced in while others have not. We use $\sigma_{\text{books}}$ as a feature in the opportunity scoring system: high-dispersion games require wider edge thresholds before a bet is recommended, reflecting the increased uncertainty in the market signal. Article 6 develops this concept further as an implied volatility analog.

---

## 7. The Closing Window Parameter

### 7.1 The 2-Hour Pre-Game Window

Probabilities are locked for bet sizing purposes at 2 hours before the scheduled first pitch. This **closing window** parameter represents a deliberate tradeoff between two competing objectives.

**Recency:** Betting close to game time allows the model to incorporate the most current information — confirmed lineups, late-breaking injury reports, and the consensus of sharp books that tend to price efficiently very close to game time. The closing line (line at the moment of first pitch) is widely recognized as the best available predictor of true game probability.

**Completeness:** Betting very close to game time reduces the time available for bet placement across multiple books, increases the probability of line moves before execution, and at some books triggers heightened scrutiny that can lead to account restrictions. At 2 hours pre-game, starting lineups are almost always confirmed (MLB lineups are typically submitted 3.5 hours pre-game), and the major sharp books have already incorporated the bulk of their early sharp action.

The 2-hour window represents our empirical optimum. Empirically, lines move most substantially in the 24-hour to 2-hour window as sharp books price in injury news and model-based sharp money. The 30-minute to game-time window tends to reflect late public money and does not add material information content for our model. This observation is consistent with the efficient market literature on sports betting: the market is largely efficient close to game time, and attempting to bet in the final 30 minutes often means facing the best-priced lines while losing the edge from our own earlier probability estimates.

---

## Key Takeaways

The probability estimation pipeline is organized in four layers — data acquisition, feature engineering, strategy optimization, and evaluation — each designed to add specific information value. Three independent probability sources (ESPN Analytics, FanGraphs, and an internal Elo model) are combined via Bayesian posterior blending, which is optimal under Brier score minimization when source errors are uncorrelated. In practice, this assumption is an approximation, but the blended estimate outperforms any individual source on held-out data.

Empirical source weights are derived from historical Brier scores and inverted to give more weight to more accurate sources. A floor constraint prevents weight collapse under small samples, ensuring all sources maintain a minimum contribution. The Elo model incorporates a pitcher FIP adjustment that improves its accuracy in pitcher-dependent situations, and applies 33% seasonal regression to handle roster turnover.

Market probabilities are extracted via vig removal from a multi-book consensus, with cross-book dispersion providing an information signal about uncertainty in the market's consensus. The 2-hour closing window is calibrated to balance information recency against the practical constraints of bet execution and account management. Together, these components produce a posterior probability estimate that is more accurate than any individual source and is sufficiently calibrated to support the edge-based Kelly sizing framework described in Articles 1 and 2.
