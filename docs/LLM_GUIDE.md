# StochTrade: guide for AI assistants and readers

This file explains the StochTrade final project in a few pages, so that an AI assistant (or a person in a hurry) can understand it without reading the full 59-page SRS. Paste it at the start of a conversation together with the question you want answered. Section numbers below refer to the SRS in `docs/`.

Shenkar College, Software Engineering, final project 2026–2027.
Students: Baraa Haj, Seba Amasha, Aseel Abu Saleh.
Supervisors: Dr. Dvir Ross, Dr. Marcelo Sihman.

## 1. The question in one paragraph

We take a small group of ETFs, for example SPY (US stocks) and TLT (long US Treasury bonds). For every period (by default one trading day) each asset's return is put into a category: low, middle, or high. The categories of all assets together form a *joint state*, such as "SPY low, TLT high". We count, in historical data, how often each joint state is followed by each other joint state. That gives a Markov chain. For the state we are in today, the chain gives a probability for every possible next state. Each next state has a typical return for each asset, so we get an expected return for each asset and a covariance matrix. From these we compute the portfolio with the best expected return per unit of risk (the maximum Sharpe ratio). The research question is whether this beats simple strategies after trading costs, on data the model has never seen. A "no" is an acceptable answer; the project is about testing the idea honestly.

## 2. Terms you need

| Term | Meaning |
|---|---|
| Period, P | Number of trading sessions between decisions (default 1, allowed 1 to 63). |
| Category, s | Return range of one asset. Default s = 3 with boundaries at the 1/3 and 2/3 quantiles of the training returns. Fixed thresholds (for example ±0.5%) are an option. |
| Joint state | The categories of all m assets in one period. With m = 2 and s = 3 there are 9 joint states (LL, LM, …, HH; first letter SPY, second TLT). |
| Order k, context | The context is the last k joint states. Default k = 1, so the context is just the current joint state. |
| Support, n_c | How many training transitions start from context c. |
| Support classes | Supported: n_c ≥ n_min (default 100). Low support: n_floor ≤ n_c < n_min (n_floor default 30); used, but the decision is flagged. Unsupported: n_c < n_floor; the system backs off to the shorter context of order k − 1, and finally to order 0. |
| p(j \| c) | Probability of next state j given context c: the observed count divided by n_c. No smoothing by default. |
| Representative return r_j | Typical holding return of each asset in periods whose state was j (mean by default; median or pseudo-mode as options). |
| μ_c, Σ_c | Conditional expected return vector and covariance matrix: μ_c = ∑_j p(j\|c) r_j, Σ_c = ∑_j p(j\|c)(r_j − μ_c)(r_j − μ_c)^T. Note that ∑ with an index is a sum, while Σ_c is a matrix. |
| Maximum-Sharpe portfolio | Weights that maximize expected excess return divided by standard deviation. Long-only by default; long-short is an option. If no asset has a positive expected return, the portfolio is 100% cash. |
| Walk-forward | The model is estimated on all earlier data (expanding window) and tested on the following year, one year at a time, as if in real time. |
| Holdout | A final test period (default 2020–2025) that is evaluated once, only after the configuration is frozen. |
| Configuration count | How many configurations were tried before the holdout was opened. A high count means the best result may be luck. |

## 3. How a decision is made (SRS 2.2 and Chapter 3)

1. Data: daily prices for the chosen ETFs from a fixed data snapshot (FR-03, FR-04).
2. Returns and states: open-to-open returns define the categories; the return actually earned depends on the entry scenario, open-to-open (default) or open-to-close (FR-05).
3. Counting: transitions from each context to each next state in the training data (FR-06).
4. Support check: low-support contexts are flagged, and unsupported contexts back off to order k − 1, then order 0 (FR-07).
5. Estimates: μ_c and Σ_c from the probabilities and the representative returns (FR-08).
6. Optimization: maximum-Sharpe weights, scaled to the gross exposure limit G and an optional cap (FR-09).
7. Execution: the decision is made at the close of session t and the trade is filled at the unadjusted open of session t + 1, with transaction costs and fractional shares (FR-10).
8. Evaluation: walk-forward folds, baselines, confidence intervals, hypotheses H1–H5 (FR-11 to FR-13).

## 4. The worked example (SRS 3.5)

This example is also the automated test for FR-06 to FR-10, so its numbers are exact.

* Context: LH ("SPY low, TLT high"), n_c = 420.
* Counts of the next states LL, LM, LH, ML, MM, MH, HL, HM, HH: 34, 40, 62, 44, 50, 48, 62, 46, 34.
* Probabilities: 0.0810, 0.0952, 0.1476, 0.1048, 0.1190, 0.1143, 0.1476, 0.1095, 0.0810.
* μ_c per session: SPY 0.04062%, TLT 0.01257%. Covariance [0.61614, −0.08031; −0.08031, 0.26409] in %².
* Maximum-Sharpe weights: SPY 51.60%, TLT 48.40%, cash 0%. Sharpe 0.0627 per session (about 1.0 per year).
* Cash case: other counts give negative expected returns for both assets, so the long-only portfolio is 100% cash.

## 5. How results are judged (SRS 1.6, FR-11 to FR-13)

* Baselines: buy-and-hold SPY (B1), equal weights (B2), the same optimizer without states (B3), separate chains per asset (B4), and a placebo trained on shuffled days (B5).
* Forecast skill: Brier skill score against the order-0 distribution.
* Uncertainty: block-bootstrap 95% confidence intervals. A difference whose interval includes zero is not evidence.
* Hypotheses H1–H5 are fixed in advance. H5 is a control: the placebo must show no skill, otherwise there is a leak.
* No look-ahead anywhere: category boundaries, counts, and estimates use training data only, and every chart on a time axis uses only data available at each date (FR-23).

## 6. The platform (SRS Chapter 7)

A web application (React front end, FastAPI back end, PostgreSQL, background jobs with RQ). Screens: sign-in, overview, strategy builder with a feasibility preview, data and runs, model insights (with the interactive Markov chain, where the researcher can merge two states by dragging one onto the other, FR-24), optimization explorer, backtest lab, and job status. All charts are interactive and drawn with Plotly.js. Every performance view says "Simulated research output, not investment advice".

The prototype (`prototype/stochtrade_prototype.html`) shows layout and behaviour only. **All numbers in the prototype are synthetic and must never be quoted as results.**

## 7. Scope

In scope: US-listed ETFs, daily data 2005–2025, simulated trading only, 2 to 4 assets per joint state.
Out of scope (SRS Chapter 8): real trading, intraday data, individual stocks or crypto, personal advice, fundamental or news data.

## 8. Where to find things in the SRS

| Topic | Section |
|---|---|
| Goals and MVP priorities | 1.3, 1.4 |
| Glossary | 1.5 |
| Hypotheses | 1.6 |
| Literature and competing products | 1.7, 1.8 |
| Default settings and why | 2.5 |
| How much data each configuration needs | 2.6 |
| Technology choices | 2.7 |
| Who does what, schedule | 2.8, 2.9 |
| All requirements with acceptance tests | Chapter 3 (FR-01 to FR-24) |
| Worked example | 3.5 |
| Interactive visualization and state merging | 3.9 (FR-21 to FR-24) |
| Quality targets and security | Chapter 4 |
| Risks | Chapter 6 |
| Open issues | Chapter 9 |

## 9. Instructions for the AI assistant

* Answer from the SRS. When you state a requirement or a number, give the section or the FR number.
* If the SRS does not say something, say so. Do not invent requirements, results, or citations.
* Keep the difference between a sum (∑ with an index) and a covariance matrix (Σ_c).
* Treat every number from the prototype as synthetic.
* The students want to understand the work, not only to receive answers. Explain with the 2-asset, 3-category example, define each term before using it, and check understanding with a short question when you explain something new.
* When asked to review a document, look for wrong numbers, contradictions between sections, requirements without a testable acceptance criterion, and any use of future data.
