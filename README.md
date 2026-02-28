
# Ex-Ante Factor-Based Tracking Error Attribution for Equity Portfolios

A complete implementation of factor-based tracking error decomposition for a 30-stock active equity portfolio. The model uses a 7-factor framework (Fama-French 5 + Momentum + Volatility) to decompose tracking error by both **factor** and **security**, with full Euler decomposition guaranteeing exact summation.

---

## Key Results

| Metric | Value |
|--------|-------|
| **Total Tracking Error** | 460.4 bps |
| Systematic TE (factor-driven) | 277.4 bps (60.3% of TE²) |
| Idiosyncratic TE (stock-specific) | 182.9 bps (39.7% of TE²) |
| Dominant Factor | Volatility (35.0% of TE²) |
| Largest Security Contributor | NVDA — 303.9 bps (66.0% of total TE) |
| Universe | 30 securities, 82 monthly observations |

---

## The 8-Step Pipeline

```
Step 1  Load Data ──────────► Module 1 output: prices, weights, factor returns
Step 2  Compute Returns ────► 85 months of prices → 84 monthly returns
Step 3  Active Weights ─────► h = portfolio weight − benchmark weight
Step 4  Factor Regressions ─► OLS: (Rᵢ − RF) ~ 7 factors → 30×7 beta matrix
Step 5  Active Exposures ───► f = Bᵀ × h (aggregate factor tilts)
Step 6  Factor Covariance ──► 7×7 covariance matrix from factor returns
Step 7  Idiosyncratic Risk ─► Σ(h² × σ²ε) for stock-specific risk
Step 8  TE Decomposition ───► Factor-level + Security-level attribution
```

---

## Mathematical Framework

### Total Tracking Error

```
TE² = f' × Σf × f  +  Σ(hᵢ² × σ²εᵢ)
       ─────────       ──────────────
       Systematic       Idiosyncratic
```

Where:
- **f** = active factor exposure vector (7×1), computed as `f = Bᵀ × h`
- **Σf** = factor covariance matrix (7×7)
- **h** = active weight vector (30×1)
- **σ²ε** = residual variance per security from Step 4 regressions

### Security Covariance Matrix

Rather than estimating a noisy 30×30 matrix directly from 82 observations, the model constructs it through the factor model:

```
V = B × Σf × Bᵀ + D
```

- **B × Σf × Bᵀ** = systematic covariance (how stocks co-move through shared factors)
- **D** = diagonal matrix of idiosyncratic variances (stock-specific noise)

This reduces parameters from 465 (direct estimation) to ~268 (factor model), producing a more stable, guaranteed positive-definite matrix.

### Marginal TE and Contribution to TE (Euler Decomposition)

```
Marginal TEᵢ = (V × h)ᵢ / Total TE       ← partial derivative: ∂TE/∂hᵢ
CTRᵢ         = hᵢ × Marginal TEᵢ          ← Euler: Σ CTRᵢ = Total TE (exact)
```

By Euler's theorem for homogeneous functions, security contributions sum **exactly** to total TE — not approximately, but with mathematical certainty. The same decomposition applies at the factor level:

```
Factor CTRⱼ = Variance_Contributionⱼ / Total TE
```

---

## Factor Attribution Results

| Factor | Active Exposure | CTR (bps) | % of TE² | Direction |
|--------|:--------------:|:---------:|:--------:|-----------|
| Vol | −0.258 | 161.2 | 35.0% | ↑ Adds risk |
| HML | −0.127 | 108.1 | 23.5% | ↑ Adds risk |
| Idiosyncratic | — | 182.9 | 39.7% | Stock-specific |
| SMB | −0.064 | 15.7 | 3.4% | ↑ Adds risk |
| Mom | −0.027 | 3.2 | 0.7% | ↑ Adds risk |
| RMW | +0.101 | 0.6 | 0.1% | ↑ Adds risk |
| Mkt-RF | −0.021 | −9.1 | −2.0% | ↓ Reduces risk |
| CMA | +0.005 | −2.3 | −0.5% | ↓ Reduces risk |
| **TOTAL** | | **460.4** | **100%** | **Sums exactly** |

---

## Top Security Contributors

| Ticker | Active Weight | Marginal TE (bps) | CTR (bps) | % of Total TE |
|--------|:------------:|:-----------------:|:---------:|:-------------:|
| NVDA | +7.65% | 3,973 | +303.9 | 66.0% |
| GOOGL | +4.29% | 1,367 | +58.7 | 12.7% |
| AAPL | +4.54% | 1,216 | +55.2 | 12.0% |
| MSFT | +4.29% | 1,172 | +50.2 | 10.9% |
| AMZN | +1.66% | 1,652 | +27.4 | 6.0% |
| AMD | −1.26% | 1,768 | −22.2 | −4.8% |

Top 5 securities contribute 107.6% of total TE, offset by diversifying positions.

---

## Repository Structure

```
├── Equity_TE_Module1_(a) Download_ticker_data.py    # Price data acquisition
├── Equity_TE_Module1_(b) Download_FF_factors.py      # Fama-French factor download
├── Equity_TE_Module1_(c) Download_other_data.py      # Treasury yields, credit spreads
├── Equity_TE_Module2_Security_Attribution.py          # 8-step TE attribution engine
├── Equity_TE_Attribution_Guide.pdf                    # Full technical write-up
├── Portfolio holdings.xlsx                            # Portfolio and benchmark weights
├── FRED data links.xlsx                               # Data source reference
└── README.md
```

### Module 1: Data Acquisition
Downloads and consolidates all required data:
- Monthly prices for 30 securities + 2 ETFs (yfinance / Twelve Data)
- Fama-French 5 factors + Momentum (Kenneth French Data Library)
- Custom Volatility factor (SPLV − SPY)
- Output: consolidated Excel workbook with 8 sheets

### Module 2: Attribution Engine
Implements the full 8-step pipeline:
- Time-series OLS regressions (30 securities × 7 factors)
- Factor covariance construction (equal-weight or EWMA)
- Security covariance via factor model: V = B × Σf × Bᵀ + D
- Euler decomposition at both factor and security level
- Output: 11-sheet Excel workbook with complete attribution

---

## The 7 Factors

| Factor | Full Name | What It Captures |
|--------|-----------|-----------------|
| **Mkt-RF** | Market Excess Return | Broad equity market risk |
| **SMB** | Small Minus Big | Size premium — small vs large caps |
| **HML** | High Minus Low | Value premium — cheap vs expensive stocks |
| **RMW** | Robust Minus Weak | Profitability — high vs low margins |
| **CMA** | Conservative Minus Aggressive | Investment — low vs high asset growth |
| **Mom** | Momentum | Winners vs losers over past 12 months |
| **Vol** | Low Volatility Minus Market | Risk appetite — defensive vs aggressive |

---

## Key Concepts

### Why Contributions Sum Exactly
TE(h) = √(h'Vh) is homogeneous of degree 1. Euler's theorem guarantees:

```
TE = Σ hᵢ × ∂TE/∂hᵢ = Σ CTRᵢ
```

This is not an empirical result — it is a mathematical property of the square root of a quadratic form.

### Why √a + √b ≠ √(a+b)
Systematic TE (277.4 bps) and Idiosyncratic TE (182.9 bps) do not add to 460.4 bps arithmetically. They combine via Pythagoras because they are uncorrelated by OLS construction:

```
Total TE = √(277.4² + 182.9²) = 460.4 bps
```

The Euler method (dividing variance contributions by total TE) avoids this problem entirely.

### Marginal TE as a Linear Approximation
Marginal TE is the delta of portfolio construction — a first derivative, valid for small active weight changes:

```
ΔTE ≈ Marginal_TEᵢ × Δhᵢ
```

For large rebalances, full revaluation (recompute h'Vh) is required because the function is nonlinear.

---

## Data Sources

| Source | Data | License |
|--------|------|---------|
| [Kenneth French Data Library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html) | FF5 factors, Momentum | Academic / free |
| [FRED](https://fred.stlouisfed.org/) | Treasury yields, credit spreads | Public domain (US Gov) |
| [Yahoo Finance](https://finance.yahoo.com/) (via yfinance) | Monthly equity prices | Personal use |
| [Twelve Data](https://twelvedata.com/) | Backup price source | API terms apply |

> **Note:** Raw price data is not included in this repository due to data provider terms of service. Run Module 1 to download all required data.

---

## Requirements

```
Python 3.8+
pandas
numpy
statsmodels
openpyxl
yfinance
```

## Usage

```bash
# Step 1: Download all data
python "Equity_TE_Module1_(a) Download_ticker_data.py"
python "Equity_TE_Module1_(b) Download_FF_factors.py"
python "Equity_TE_Module1_(c) Download_other_data.py"

# Step 2: Run attribution
python Equity_TE_Module2_Security_Attribution.py
```

Output: `Equity_TE_Attribution.xlsx` with 11 sheets of attribution results.

---

## Technical Documentation

See **Equity_TE_Attribution_Guide.pdf** for the complete technical write-up including:
- Mathematical derivations for all 8 steps
- Regression interpretation with real examples (NVDA, BAC, LLY)
- Security-level and factor-level attribution tables
- PM action plan for reducing tracking error
- Statistical significance analysis (t-statistics across all 30 securities)

---

## Related Work

This project is part of a broader multi-asset risk analytics portfolio:
- **Fixed Income TE Attribution** — KRD-based tracking error with callable bond pricing
- **Multi-Asset VaR** — Filtered Historical Simulation with GARCH conditional volatility
- **Callable Bond Pricing** — QuantLib implementation with OAS and negative convexity analysis

The shared mathematical framework across all modules is the quadratic form: **f' × Σ × f** — the same structure whether f represents equity factor exposures, KRD exposures, or VaR sensitivities.
