# theta_decay_analysis
# Continuous Time Decay & The Expiration Singularity (Theta Optimization)

A quantitative derivatives module designed to simulate the non-linear path of capital degradation (**Theta Time Decay**) for an At-The-Money (ATM) option contract over its remaining lifespan.

## Methodology & Financial Calculus

The engine solves the continuous partial derivative of the Black-Scholes-Merton (BSM) equation with respect to time (T):

\[\Theta_{\text{Call}} = -\frac{S \cdot n(d_1) \cdot \sigma}{2 \sqrt{T}} - r \cdot K \cdot e^{-rT} \cdot N(d_2)\]

*   **Vector Scaling:** While the raw calculus evaluates decay across a full annualized year (T=1.0), this pipeline applies a localized divisor (Θ / 365.0) to scale the output down to an operationally useful 24-hour calendar tracking unit.

---

## The Expiration Singularity: Handling Boundary Anomalies

When plotting the complete horizon down to zero days remaining, the resulting chart produces an aggressive, near-vertical terminal spike directly at the expiration boundary (T → 0). 

### 1. The Mathematical Rationale
This dramatic curve behavior is mathematically correct. Look at the primary term of the numerator:
\[\lim_{T \to 0} \left( -\frac{S \cdot n(d_1) \cdot \sigma}{2 \sqrt{T}} \right) = -\infty\]
Because the annualized time vector (T) sits inside a square root in the denominator, dividing any constant by a value approaching zero forces the limit to blow up to infinity.

### 2. The Trading Intuition
An At-The-Money contract sits exactly on the strike price knife-edge. On its final trading session, its remaining extrinsic "time value" collapses completely in a matter of hours. The option's pricing distribution behaves like a delta function—the slightest tick dictates whether the contract expires fully preserved or completely worthless.

### 3. Data Engineering Resolution
While mathematically accurate, this terminal infinity spike skews the vertical visual scale, flattening the critical trajectory details. To preserve chart readability, the Python visualization layer implements a data quality gate:
```python
# Filters out the final terminal step (Day 0) to maintain uniform axis scaling
clean_plot_df = decay_df[decay_df['Days_To_Expiration'] > 0]
```

---

## Insights
<img width="3600" height="1950" alt="theta_decay_shock_dashboard" src="https://github.com/user-attachments/assets/31bf0335-6e66-4880-85d7-c819a52362cb" />

This visual curve provides concrete, rule-based execution logic for derivatives portfolios:

1.  **The Option Buyer's Cliff (Exit Rules):** The decay trajectory is relatively flat from Day 90 down to Day 45. However, at **45 days to expiration**, the curve hits an inflection point and accelerates exponentially. Options buyers must systematically close or roll long positions before this 45-day threshold to avoid severe capital drag.
2.  **The Option Seller's Sweet Spot (Entry Rules):** Symmetrical to the buyer's risk, net-sellers of premium maximize their mathematical edge by entering short positions inside the **45-day to 15-day window**. This range harvests the steepest portion of the decay curve while closing out before terminal settlement risks multiply.

