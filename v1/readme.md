# Self-Calibrating Transport Engine (C++)

A high-performance quantitative financial valuation engine written in standard, zero-overhead C++. The engine applies **Optimal Transport (OT) Theory** to evaluate financial options chains, calculating the **1st Wasserstein Distance (Earth Mover's Distance)** between empirical risk-neutral market distributions and idealized theoretical log-normal benchmarks to measure real-time market strain anomalies.

---

## Technical & Mathematical Architecture

### 1. Data Ingestion & Memory Architecture
* **`HistoryRow` & `OptionRow` Layouts:** Designed to optimize cache line access by structure alignment during stream parsing (`std::ifstream`) of time-series data and derivative matrix cross-sections.
* **Hermetic Execution Primitives:** Implements custom mathematical approximations for `normal_cdf` and `normal_pdf` using the complementary error function (`std::erfc`), bypassing massive external library dependencies for predictable execution environments.

### 2. High-Frequency Volatility Estimation (Garman-Klass)
Rather than executing slow, lag-heavy close-to-close standard deviation measurements, the engine deploys the intra-day efficient **Garman-Klass (GK)** open-high-low-close estimator:

\[\sigma_{ohlc} = \sqrt{\frac{252}{n}\sum_{i=1}^{n}\left[0.5\left(\ln \frac{H_{i}}{L_{i}}\right)^{2}-(2\ln 2-1)\left(\ln \frac{C_{i}}{O_{i}}\right)^{2}\right]}\]

This effectively isolates systemic historical market footprints directly from raw price feeds.

### 3. Latent Macroeconomic Parameter Extraction
The pipeline utilizes an adjusted **Put-Call Parity linear regression** cross-section to solve for unobservable baseline values—specifically the risk-free rate (\(r\)) and asset dividend drift (\(d\)):

\[C(K) - P(K) = S_{0}e^{-dT} - Ke^{-rT}\]

By setting \(Y = C - P\) and \(X = -K\), the system extracts \(\text{slope} = e^{-rT}\) and \(\text{intercept} = S_{0}e^{-dT}\). The metrics are scaled safely over time to maturity (\(T\)) and bound by architectural guardrails to prevent numerical overflows:

\[r = -\frac{\ln(\text{slope})}{T}, \quad d = -\frac{\ln(\text{intercept} / S_{0})}{T}\]

### 4. Newton-Raphson Volatility Inversion
To trace the baseline volatility priced by market makers, the module targets the **At-The-Money (ATM)** strike and solves the inverse Black-Scholes pricing problem using a Newton-Raphson iteration loop:

\[\sigma_{n+1} = \sigma_{n} - \frac{f(\sigma_{n}) - \text{Target Price}}{\text{Vega}(\sigma_{n})}\]

The partial derivative (\(\text{Vega}\)) is calculated numerically via forward differences (\(h = 10^{-4}\)) ensuring fast, 64-bit precision convergence.

### 5. Dual Distribution Space Mapping
* **Empirical Vector (\(\mu_T\)):** Leverages the classical **Breeden-Litzenberger theorem**, stating that the second derivative of call options prices with respect to strikes yields the exact risk-neutral probability density function:
  
  \[p(K) = e^{rT}\frac{\partial^{2}C}{\partial K^{2}}\]
  
  The code tracks this with a discrete second-difference numerical stencil over the strike chain. It applies an institutional liquidity weight using log-transformed option Open Interest to scale real capital commitments over localized spread noise:
  
  \[\mu_{T}[i] \propto \frac{\partial ^{2}C}{\partial K^{2}} \cdot \left(1.0 + 0.05\ln(\text{Open Interest} + 1)\right)\]

* **Theoretical Vector (\(\nu_t\)):** Generates a smooth, benchmark log-normal distribution mapped over the extracted drift trends and ATM implied volatility.

### 6. 1D Optimal Transport Integration
By aggregating the probability vectors into **Cumulative Distribution Functions (CDFs)** (\(F_\mu\) and \(F_\nu\)), the engine calculates the **1st Wasserstein Distance**. In a single dimension, the minimum energy required to map the empirical mass into the baseline simplifies to the integrated area between their CDF curves:

\[\mathcal{W}_{1}(\mu, \nu) = \int_{-\infty}^{\infty} \vert{}F_{\mu}(x) - F_{\nu}(x)\vert{} \, dx\]

---

## File Structure & Requirements

The system processes text configuration inputs localized within its running directory. Ensure the following files exist before execution:

### 1. `market_history.txt`
Contains historical OHLCV data. The engine uses the final row's `close` price as the current spot price reference (\(S_0\)).
```text
date,open,high,low,close,volume
2026-11-02,201.20,203.50,199.80,202.10,1250000
...
```

### 2. `option_chain.txt`
Contains the structural option strikes and pricing spreads.
```text
call_oi,call_ltp,strike,put_ltp,put_oi
500,13.65,190.00,0.35,1200
...
```

---

## Compilation and Execution

The pipeline is written in header-less standard C++ with no external runtime dependencies. Compile using any modern C++11 compliant compiler with high optimization flags:

```bash
# Compilation using g++
g++ -O3 main.cpp -o transport_engine

# Execution
./transport_engine
```

### Sample Pipeline Output
```text
==================================================
      SELF-CALIBRATING TRANSPORT ENGINE (C++)     
==================================================
Extracted System Parameters Matrix:
  Current Spot (S0):             202.500000
  Historical OHLC Vol (Annual):  17.814319 %
  Extracted Risk-Free Rate (r):   5.000000 %
  Extracted Implied Drift (d):   0.000000 %
  Extracted ATM Implied Vol:     20.000000 %

Pipeline Output Targets:
  Robust Expected Value (Mean):  202.483120
  True Wasserstein 1D Distance:  0.045182
  Dynamic Transport Distortion:  1.000741
==================================================
```
