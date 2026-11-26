# StochTrade

**A joint-state Markov research platform for ETF portfolio allocation**
Final project, B.Sc. in Software Engineering, Shenkar College, Faculty of Engineering (2026–2027).

| | |
|---|---|
| Team | Seba Amasha ([@sebaamasha](https://github.com/sebaamasha)) · Baraa Haj ([@Baraa40](https://github.com/Baraa40)) · Aseel Abu Saleh |
| Supervisors | Dr. Dvir Ross · Dr. Marcelo Sihman |
| Current stage | Software Requirements Specification (November 2026) |

## What the project does

StochTrade tests whether the **joint** recent behaviour of a small group of ETFs (for example SPY and TLT) carries usable information about their next-period returns.

1. Each asset's period return is assigned to a category (by default low, middle, or high, from training quantiles). The categories of all assets form a **joint state**.
2. An empirical **Markov chain** counts how often each joint state follows each context and uses the observed frequencies as the next-state distribution.
3. Each next state carries a representative return vector, so the chain gives a **state-conditional expected return vector and covariance matrix**.
4. In every state the system finds the portfolio with the **highest expected excess return per unit of risk** (long-only by default, long-short as an option) and holds cash when no positive expectation exists.
5. A **walk-forward backtest** with costs compares the strategy with baselines (buy-and-hold, 1/N, the same optimizer without states, separate chains per asset, and a placebo) on a locked holdout, with block-bootstrap confidence intervals and pre-registered hypotheses.

Predictability is treated as a hypothesis to test; negative results are reported.

## Documents

| Document | File |
|---|---|
| Software Requirements Specification (SRS) | [`docs/StochTrade_SRS_3.pdf`](docs/StochTrade_SRS_3.pdf) |
| Guide for AI assistants and readers | [`docs/LLM_GUIDE.md`](docs/LLM_GUIDE.md) — paste it into an AI assistant to explain the project |
| Software Design Document (SDD) | due 22 January 2027 |

## Screen prototype

- **Open online:** https://sebaamasha.github.io/StochTrade/prototype/stochtrade_prototype.html
- **Open offline:** download [`prototype/stochtrade_prototype.html`](prototype/stochtrade_prototype.html) and double-click it. It is a single file with no server; the chart library (Plotly.js 2.35.2) is embedded.
- **All numbers in the prototype are synthetic.** They illustrate layout and behaviour only and must not be cited as results.

Screens: Overview (SC-02), Strategy builder with feasibility heatmap and draggable category thresholds (SC-03), Data and runs with saved studies (SC-04), Model insights with the **interactive Markov chain** — click a state to see its distribution, weights, and an explanation — plus the transition heatmap, state timeline, and chain quality (SC-05), Optimization explorer with the efficient frontier (SC-06), Backtest lab with equity, drawdown, rolling Sharpe, and the hypotheses forest plot (SC-07), and Job status (SC-08). Every chart is interactive, and every time-axis chart is computed point-in-time (SRS FR-21 to FR-24); in the Markov chain view, drag one state onto another to merge them and compare the merged chain with the original.

## Repository structure

| Folder | Contents |
|---|---|
| `docs/` | Project documents: SRS now; SDD, validation report, and final report later |
| `prototype/` | Interactive screen prototype (single HTML file) |
| `backend/` | Planned: Python 3.12 and FastAPI — data pipeline, Markov model, optimization, backtester (from milestone M1) |
| `frontend/` | Planned: React interface with Plotly.js charts (from milestone M2) |
| `tests/` | Planned: automated tests, including the numeric acceptance criteria of SRS Chapter 3 |

## Technology (SRS Section 2.7)

Python 3.12, FastAPI, FastAPI Users, PostgreSQL, RQ with Redis, NumPy, pandas, SciPy, CVXPY, React, Plotly.js, exchange_calendars, Docker Compose, and GitHub Actions.

## Milestones

| Milestone | Date |
|---|---|
| SRS signed | 27 Nov 2026 |
| M1 — data foundation | 25 Dec 2026 |
| SDD signed | 22 Jan 2027 |
| M2 — model core | 19 Feb 2027 |
| Alpha review (MVP) | 26 Mar 2027 |
| Beta review | 4 Jun 2027 |
| Validation report | 25 Jun 2027 |
| Final project report | 31 Aug 2027 |

## Working rules

- Every change goes through a pull request reviewed by the module's reviewer (SRS Section 2.8).
- Tests run on every push; every acceptance criterion with numbers becomes an automated test.
- No secrets or API keys are committed; they are read from environment variables.
