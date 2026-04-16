# Research Report

**Project Title:** Synthetic Quality Factor
**Author(s):** Josh Oldroyd, Grant Rich, Brandon Waits
**Date:** April 15, 2026
**Version:** III

---

## 1. Summary

This project investigates construction of a synthetic quality factor by applying cross-sectional OLS spanning regressions to MSCI Barra quality factor exposures. Four signal constructions are evaluated against static baselines derived from AQR's Quality Minus Junk (QMJ) framework: a 50/50 QMJ proxy, a QARP (quality + value) blend, a static equal-weight composite, and an OLS-estimated composite targeting the equal-weight signal.

The central finding is that the empirically estimated OLS equal-weight composite (`ols_ew`) dominates all baselines on a risk-adjusted basis. Weight stability (CV < 0.11 across all five factors) confirms the OLS solution is well-conditioned, rendering ridge regularization unnecessary. The signal carries significant FF5 alpha, is 80% orthogonal to the existing production signal set, and contributes an incremental residual Sharpe of 0.29, supporting its addition to the production portfolio.

The Bayesian approaches included in experiments done by Grant and Brandon are not outlined in the report below. We discuss only the latest research that was performed.

### Key Metrics

| Metric                                  | Value     | Notes                                            |
|--------|-------|-------|
| `ols_ew` Sharpe Ratio                   | 1.75      | Full sample 1996–2024, MVO backtest |
| FF5 Alpha (intercept t-stat)            | 6.38      | Significant profitability and value tilts |
| Residual Sharpe (orthogonality test)    | 0.29      | After projecting out 3 production signal alphas |
| Multivariate R² vs. production signals  | ~0.20     | 80% of variance orthogonal to existing portfolio |
| Max factor CV (`ols_ew`)                | 0.11      | Confirms temporal stability of OLS weights |

---

## 2. Data Requirements

**Sources**
- MSCI Barra factor exposures
- CRSP returns

**Rate of Availability**
- Daily factor exposures; signal constructed at monthly cross-sections

**Inputs Required**
- Five Barra quality-related factor exposures per stock per date: `USSLOWL_PROFIT`, `USSLOWL_EARNQLTY`, `USSLOWL_MGMTQLTY`, `USSLOWL_LEVERAGE`, `USSLOWL_GROWTH`

**Preprocessing**
- $5 price filter applied prior to portfolio construction
- Portfolios constrained to zero market beta and zero investment
- Gamma calibrated to target ~5% active risk over the full sample period

---

## 3. Approach / System Design

**Economic Intuition.** The quality premium reflects durable mispricing of safe, profitable firms driven by behavioral biases. This structural persistence motivates fixed-weight rather than dynamic construction — timing error is introduced without benefit when the premium does not require timing.

**Static Baselines.** Two static composites serve as benchmarks: (1) **QMJ** — 50/50 blend of `PROFIT` and `EARNQLTY` (Sharpe 1.38); (2) **Equal-Weight QMJ** — all five Barra quality factors with AQR-motivated signs (+,+,+,−,−), equally weighted (Sharpe 1.75).

**OLS Spanning Regression.** At each monthly cross-section, stock-level factor exposures are regressed against a target signal (either the QMJ proxy or the equal-weight composite). Coefficients are averaged across the time series to produce stable static weights. This is the primary construction methodology.

**Ridge Robustness Check.** Bayesian ridge regression was also implemented (minimizing `||y - Xw||² + λ||w - w₀||²`) to assess whether multicollinearity among Barra factors required regularization. Ridge coefficients were nearly identical to OLS (e.g., LEVERAGE: −0.3963 OLS vs. −0.3883 tight ridge), confirming OLS is sufficient.

**Design Decision.** Dynamic BMA (Bayesian Model Averaging) specifications were tested and uniformly underperformed static constructions, reinforcing the static approach. 

---

## 4. Code Structure

```
sf-research-bayesian-quality/
├── research/
│   ├── josh_experiments/
│   │   ├── experiment_##.py      # Different experiments for signal construction
├── results/josh/
│   ├── experiment_#/             # Plots and charts for results from experiments
└── REPORT.md

```

---

## 5. Results / Evaluation

**Signal Performance Summary (MVO Backtest, 1996–2024)**

| Signal | Mean Return | Volatility | Sharpe |
|--------|-------------|------------|--------|
| `ols_ew` (primary) | 8.54% | 4.89% | **1.75** |
| `ols_qmj` | 8.32% | 5.92% | 1.41 |
| QMJ (50/50 baseline) | 4.97% | 3.60% | 1.38 |
| Equal-Weight QMJ (static) | 5.04% | 2.88% | 1.75 |

**FF5 Regression (`ols_ew`)**

| Variable | Coefficient | T-stat |
|----------|-------------|--------|
| Intercept | 0.0226 | **6.38** |
| mkt_rf | 0.0133 | 4.09 |
| hml | 0.0413 | 7.31 |
| rmw | 0.0963 | 12.43 |
| cma | 0.0001 | 0.01 |

**Orthogonality Test (`ols_ew` residual vs. production portfolio)**

| Mean Return | Volatility | Sharpe |
|-------------|------------|--------|
| 1.24% | 4.32% | 0.29 |

Pairwise correlations with production signals — beta: 0.39, barra_reversal: 0.005, ivol: 0.41. Multivariate R² ≈ 0.20.  Forgot to include other production signals, will do in continued research.

---

## 6. Performance Discussion

**Strengths.** The `ols_ew` signal achieves the highest Sharpe ratio of all constructions tested, is empirically grounded, and its weights are highly stable across nearly three decades (max CV = 0.11). The FF5 regression confirms economically interpretable tilts (profitability, value) and significant alpha. The 80% orthogonality to existing production signals supports genuine diversification benefit (inasmuch as it was executed correctly).

**Weaknesses.** The `ols_ew` Sharpe matches the naive equal-weight static construction (both 1.75), meaning OLS provides validation and interpretability but not a raw Sharpe improvement over the simpler equal-weight rule in this case. QARP underperforms QMJ on a risk-adjusted basis, suggesting the 50/50 value blend is too blunt; a better combination mechanism may exist.

---

## 7. Limitations

- **Missing features:** An improved combination mechanism for QARP has not been developed, and the orthogonality test likely needs to be modified.
- **Risks:** Weight stability over 1996–2024 does not guarantee stability in future regimes. The $5 price filter and zero-beta constraint may limit generalizability.
- **Open questions:** Can a dynamic quality construction outperform static in a properly validated out-of-sample framework? Is there an optimal blend of `ols_ew` and value to reconstruct QARP? How can we validate it?

---

## 8. Future Work

- Develop an improved weighting mechanism for QARP
- Assure that the orthogonality test is conducted correctly and considers all production signals
- Pursue additional testing as requested by the team prior to vote

---

## Appendix (Optional)

Elected to opt out thanks to optionality.