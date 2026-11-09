<!-- PROJECT BANNER -->
<br />
<div align="center">
  <h2 align="center">🧮 Quantitative Transport & Volatility Engine</h2>

  <p align="center">
    A self-calibrating C++ engine mapping asset variance matrices to option surface distributions using Garman-Klass volatility, dual OLS optimizations, and empirical transport grids.This document provides a comprehensive structural breakdown of the quantitative financial algorithm
implemented in the C++ framework. The system extracts underlying risk parameters from combined historical
spot distributions and real-time option chain matrices. It then calculates the empirical risk-neutral probability
density to evaluate non-linear structural distortion against classical log-normal asset distributions
    <br />
    <br />
    <a href="#-core-components">Project Bundle</a>
    ·
    <a href="#-architectural-pipeline">System Pipeline</a>
    ·
    <a href="#-mathematical-framework">Mathematical Core</a>
    ·
    <a href="#-data-structures">Data Schemas</a>
  </p>
</div>

<!-- BADGES -->
<div align="center">

[![Language](https://shields.io)](https://cppreference.com)
[![Dependencies](https://shields.io)](https://cppreference.com)
[![Precision](https://shields.io)](#-mathematical-framework)

</div>

---

## 📖 Table of Contents
- [About the Project](#-about-the-project)
- [📁 Core Components](#-core-components)
- [🔬 Architectural Pipeline](#-architectural-pipeline)
- [🧮 Mathematical Framework](#-mathematical-framework)
- [🗃️ Data Structures & File Layouts](#%EF%B8%8F-data-structures--file-layouts)
- [🚀 Compilation & Execution](#-compilation--execution)

---

## 🧐 About the Project

This repository provides a high-throughput, native quantitative execution environment built entirely from scratch in pure C++. The architecture reads raw historical underlying matrices and multi-strike option surfaces to extract risk-free rates, forward drift values, and implied volatility layers. It then leverages regularized finite differences combined with optimal transport metrics to extract robust price expectations (\(V^{*}\)) and tail distortion metrics without relying on any external BLAS, LAPACK, or statistical runtime software.

---
[market_history.txt]  --> Garman-Klass Volatility Estimation --> Spot Price Extraction (S0)
                                                                       |
[option_chain.txt]   --> Put-Call Parity Regression -------------------> Extracted Rates (r, d)

                               |                                       |
                     Breeden-Litzenberger Formula            Black-Scholes Inversion

                               |                                       |
                    Empirical Profile (mu_T)                Theoretical Profile (nu_t)
                               \                                      /
                                \--> 1D Wasserstein Integration <----/
                                               |
                                     [Console Dashboards]
## 📁 Core Components

All key objects are bundled systematically within your `generated/` workspace folder:

* **`engine.cpp`**: The full C++ production script utilizing standard headers (`<vector>`, `<cmath>`, `<fstream>`, `<sstream>`) to compute native erfc-based distributions, OLS regressions, and finite difference matrices.
* **`market_history.txt`**: Historical Open, High, Low, Close, and Volume (OHLCV) timeseries data used to seed path variance baselines.
* **`option_chain.txt`**: The multi-strike option pricing chain tracking Open Interest (OI) metrics and Last Traded Prices (LTP).
* **`output.txt`**: The real-world terminal execution readout displaying exact parameter extractions and robust target valuations.
* **`idea.pdf`**: Academic research paper detailing the underlying physics-to-finance integration framework with modern citations.

---

## 🔬 Architectural Pipeline

The codebase executes a precise three-stage pipeline to arrive at the targeted robust properties:

<!-- PROJECT BANNER -->
<br />
<div align="center">
  <h2 align="center">🧮 Quantitative Transport & Volatility Engine</h2>

  <p align="center">
    A self-calibrating C++ engine mapping asset variance matrices to option surface distributions using Garman-Klass volatility, dual OLS optimizations, and empirical transport grids.
    <br />
    <br />
    <a href="#-core-components">Project Bundle</a>
    ·
    <a href="#-architectural-pipeline">System Pipeline</a>
    ·
    <a href="#-mathematical-framework">Mathematical Core</a>
    ·
    <a href="#-data-structures">Data Schemas</a>
  </p>
</div>

<!-- BADGES -->
<div align="center">

[![Language](https://shields.io)](https://cppreference.com)
[![Dependencies](https://shields.io)](https://cppreference.com)
[![Precision](https://shields.io)](#-mathematical-framework)

</div>

---

## 📖 Table of Contents
- [About the Project](#-about-the-project)
- [📁 Core Components](#-core-components)
- [🔬 Architectural Pipeline](#-architectural-pipeline)
- [🧮 Mathematical Framework](#-mathematical-framework)
- [🗃️ Data Structures & File Layouts](#%EF%B8%8F-data-structures--file-layouts)
- [🚀 Compilation & Execution](#-compilation--execution)

---

## 🧐 About the Project

This repository provides a high-throughput, native quantitative execution environment built entirely from scratch in pure C++. The architecture reads raw historical underlying matrices and multi-strike option surfaces to extract risk-free rates, forward drift values, and implied volatility layers. It then leverages regularized finite differences combined with optimal transport metrics to extract robust price expectations (\(V^{*}\)) and tail distortion metrics without relying on any external BLAS, LAPACK, or statistical runtime software.

---

## 📁 Core Components

All key objects are bundled systematically within your `generated/` workspace folder:

* **`engine.cpp`**: The full C++ production script utilizing standard headers (`<vector>`, `<cmath>`, `<fstream>`, `<sstream>`) to compute native erfc-based distributions, OLS regressions, and finite difference matrices.
* **`market_history.txt`**: Historical Open, High, Low, Close, and Volume (OHLCV) timeseries data used to seed path variance baselines.
* **`option_chain.txt`**: The multi-strike option pricing chain tracking Open Interest (OI) metrics and Last Traded Prices (LTP).
* **`output.txt`**: The real-world terminal execution readout displaying exact parameter extractions and robust target valuations.
* **`idea.pdf`**: Academic research paper detailing the underlying physics-to-finance integration framework with modern citations.

---

## 🔬 Architectural Pipeline

The codebase executes a precise three-stage pipeline to arrive at the targeted robust properties:


1. **Path Volatility Ingestion**: Ingests historical daily candles to calculate an unbiased annualized path variance using open, high, low, and close points.
2. **Put-Call Parity OLS Solver**: Translates call-minus-put strike arrays using linear regression to isolate risk-free rates (r) and continuous dividend/drift coefficients (d) natively.
3. **Transport Grid Alignment**: Solves a finite difference second derivative across strikes to establish the risk-neutral distribution, mapping it against a lognormal target model via distribution transportation layers.

---

## 🧮 Mathematical Framework

### 1. Localized Realized Volatility (Garman-Klass)
Utilizes the intraday path range to produce a minimum-variance estimator annualized over a standard 252-day business matrix:
\[\sigma_{ohlc} = \sqrt{\frac{252}{N} \sum_{t=1}^{N} \left[ 0.5 \left( \ln \frac{H_t}{L_t} \right)^2 - (2\ln 2 - 1) \left( \ln \frac{C_t}{O_t} \right)^2 \right]}\]

### 2. Native Parameter Calibration (OLS)
Applies linear regression to the option surface array by mapping strike differences against premium differences (\(Y_i = C_i - P_i\), \(X_i = -K_i\)). The continuous system rate and drift are then solved analytically:
\[r = -\frac{\ln(\text{slope})}{T}, \quad d = -\frac{\ln(\text{intercept} / S_0)}{T}\]

### 3. Implied Volatility Engine (Newton-Raphson)
Implements an optimized root-finding method leveraging a native error function complement (erfc) to reverse-engineer ATM Implied Volatility (σ):
\[\sigma_{n+1} = \sigma_n - \frac{BS_{call}(\sigma_n) - C_{target}}{\mathcal{V}(\sigma_n)}\]

### 4. Transport Distribution & Robust Expectation (\(V^{*}\))
Extracts empirical risk-neutral mass (\(\mu_T\)) by applying a regularized finite difference operator weighted by open interest log-growth, then establishes a lognormal target distribution (\(\nu_t\)) to track divergence boundaries:
\[\mu_{T, i} = \max\left(\frac{C_{i+1} - 2C_i + C_{i-1}}{(\Delta K)^2}, 0\right) \cdot e^{rT} \cdot (1 + 0.05\ln(\text{OI}_{total} + 1))\]
\[V^* = \sum_{i=1}^{N} \mu_{T, i} \cdot K_i\]

---

## 🗃️ Data Structures & File Layouts

### Expected File Formats (Comma-Separated)

#### 1. `market_history.txt`
```text
Date,Open,High,Low,Close,Volume
2026-10-01,150.00,152.50,149.10,151.20,5000000
2026-10-02,151.50,153.00,150.25,152.80,6200000
```

#### 2. `option_chain.txt`
```text
call_oi,call_ltp,strike,put_ltp,put_oi
12500,15.20,140.00,0.45,8400
18400,8.40,150.00,3.10,22100
```

---

## 🚀 Compilation & Execution

Because the implementation uses pure C++ standard library parts, compilation is fast and cross-platform.

### Linux / macOS
```bash
# Navigate to the workspace containing your source and text matrices
cd generated

# Compile with high optimization and vectorization switches
g++ -O3 -std=c++17 engine.cpp -o transport_engine

# Execute the binary
./transport_engine
```

### Windows (MSVC Core Developer Command Prompt)
```cmd
cl /EHsc /O2 /std:c++17 engine.cpp /fe:transport_engine.exe
transport_engine.exe
```

---

## 📊 Live Telemetry Display

Upon successful processing, the runtime screen output logs directly to the console in the following structure:

```text
==================================================
      SELF-CALIBRATING TRANSPORT ENGINE (C++)     
==================================================
Extracted System Parameters Matrix:
  Current Spot (S0):             152.800000
  Historical OHLC Vol (Annual):  24.156200 %
  Extracted Risk-Free Rate (r):   4.521800 %
  Extracted Implied Drift (d):    1.204500 %
  Extracted ATM Implied Vol:     22.841500 %

Pipeline Output Targets:
  Robust Cost of Transport (V*): 153.142500
  Implied T' Distortion Ratio:   1.082000
==================================================
```

---

## 📄 License

Distributed under the MIT License. Built safely without commercial external codebases.
