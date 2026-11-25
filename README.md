# StochTrade

A joint-state Markov research platform for ETF portfolio allocation.
Final project, B.Sc. Software Engineering, Shenkar College (2026–2027).

**Team:** Baraa Haj, Seba Amasha, Aseel Abu Saleh
**Supervisors:** Dr. Dvir Ross (research and methodology), Dr. Marcelo Sihman (software engineering)

> StochTrade is a research tool. All results are simulated research outputs, not investment advice.

## Research question

Does the joint recent behaviour of a small group of ETFs (for example SPY and TLT) carry usable information about their next-period returns, and does a portfolio built from that information perform better, after costs, than portfolios that ignore it?

## How the system works

1. **Data.** Daily prices are retrieved for the selected ETFs, validated, and stored as an immutable snapshot that records its source, terms of use, and version.
2. **States.** Each asset's open-to-open return over a period of P sessions is assigned to a category (by default terciles of its training returns: low, middle, high). The categories of all assets together form a joint state, such as "SPY low, TLT high".
3. **Markov model.** Transitions from each context to each next joint state are counted. The probabilities are the empirical frequencies, with no smoothing by default; contexts with too few transitions back off to shorter contexts, and every estimate is shown with its support.
4. **Conditional return and risk.** The next-state probabilities and each state's representative holding return give a conditional expected return vector and covariance matrix.
5. **Optimization.** The portfolio with the highest expected excess return per unit of risk is chosen: long-only by default, long-short as an option, with cash when no positive expectation exists.
6. **Evaluation.** A walk-forward backtest with transaction costs compares the strategy with baselines (buy-and-hold, 1/N, the same optimizer without states, separate chains per asset, and a shuffled-day placebo) on a locked holdout segment, with block-bootstrap confidence intervals and pre-registered hypotheses.

The full specification is in the SRS.

## Repository structure

| Folder | Contents |
|---|---|
| `docs/` | Project documents: the SRS, and later the SDD and the test and evaluation reports |
| `prototype/` | `stochtrade_prototype.html`: interactive screen prototype |

The application code will be added after the SDD (due 22 January 2027) in this planned layout:

| Folder (planned) | Contents |
|---|---|
| `backend/` | Python 3.12 and FastAPI: data, model, optimization, backtest engine, API |
| `frontend/` | React web interface |
| `tests/` | Unit and acceptance tests; the worked examples of the SRS are test fixtures |

## Prototype

**Live version:** https://sebaamasha.github.io/StochTrade/prototype/stochtrade_prototype.html

The prototype is a single HTML file with plain JavaScript and no server. To run it locally, download `prototype/stochtrade_prototype.html` and open it in a browser (Chrome, Edge, or Firefox).

| Screen | What it shows |
|---|---|
| SC-01 Sign in | Sign-in and registration forms |
| SC-02 Overview | Current context, latest weights, comparison with baselines |
| SC-03 Strategy builder | All experiment settings, feasibility preview, category-split preview, validation |
| SC-04 Data and runs | Snapshot details and the list of runs |
| SC-05 Model insights | Transition table, chain quality, forecast skill |
| SC-06 Optimization explorer | Conditional estimates and the resulting portfolio, long-only or long-short |
| SC-07 Backtest lab | Equity curves, metrics, hypotheses report, folds, holdout lock |
| SC-08 Job status | Progress, cancellation, and failure reasons |

The values in the Overview and Optimization Explorer reproduce the worked example of SRS Section 3.5 (weights SPY 51.60% and TLT 48.40%).

**All other numbers in the prototype are synthetic.** They illustrate layout and behaviour only and must not be cited as results.

## Working conventions

- Each module has one owner and one reviewer (SRS Section 2.8).
- Work is done on feature branches and merged into `main` through a pull request approved by the reviewer.
- Every acceptance criterion with numbers in the SRS becomes an automated test.

## Status and milestones

| Milestone | Date |
|---|---|
| SRS submitted for supervisor approval | November 2026 |
| M1: data foundation | 25 December 2026 |
| SDD | 22 January 2027 |
| M2: model core | 19 February 2027 |
| Alpha review (MVP) | 26 March 2027 |
| Beta review | 4 June 2027 |
| Final report | 31 August 2027 |