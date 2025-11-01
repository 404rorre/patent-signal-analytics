![license](https://img.shields.io/badge/license-MIT-green?style=flat) ![Python](https://img.shields.io/badge/Python-3.12-orange?style=flat)


# Forecasting & Predictive Analytics
---
>**TL;DR**
>Time-series analysis & predictive modeling. Tests whether innovation metrics predict
>future revenue growth and highlight disruptive companies, using patent & financial data (SATCOM case study).
---

Concept about analysing disruptive companies using financial and patent datasets.

## Overview

This repository contains an applied concept examining whether patent activity 
predicts future revenue performance in satellite communications. The analysis emphasizes 
transparent methodology, limitations, and actionable insights for strategic planning.

---

## 1. Technology Gap KPI Framework

**File:** `technology_gap.ipynb`

### Motivation

R&D spending is a core capital allocation decision, but many firms, especially private ones, don’t disclose it consistently. This project tests whether patent activity, as a proxy for disruptive innovation intensity, predicts revenue growth. It also demonstrates how to operationalize patent signals across public and private SATCOM companies.

### Research Question

Does normalized patent filing activity predict subsequent revenue growth? If so, what time 
lags are most predictive, and do patterns vary across value chain positions (upstream, 
midstream, downstream)?

### Data

| Source | Coverage | Records | Notes |
|--------|----------|---------|-------|
| Orbis (Bureau van Dijk) | European companies + major non-EU operators | 95 companies | NACE Rev. 2: 6130 Satellite Communication - raw data excluded|
| Justia Patents | US-registered patents | 4,512 filings | Space satellite patents (US Class 455/12.1). Public access via Justia. |
| Manual consolidation | SATCOM value chain segments | 3 categories | Proprietary classification |

**Data Availability:** Raw data is subject to institutional licensing agreements and is not 
included in this repository. The analysis pipeline is fully reproducible using similar 
financial and patent datasets.

**Data Limitations:** Orbis completeness improves significantly post-2016; pre-2015 records 
show gaps for private companies. Patent data skews US/EU; emerging market filers underrepresented.

**Data Anonymization:** Company names have been anonymized (random codes) for Orbis licensing compliance.

### Methodology

#### Step 1: Data Integration
- Matched company names across Orbis and patent databases (fuzzy matching with manual verification)
- Aggregated annual revenue and patent counts by calendar year
- Classified companies into value chain segments based on primary business description

#### Step 2: KPI Construction (Worked Example)

Fictitious company example for a 3-year rolling window (2019–2021):

- **Financial data (Annual Growth Rate - AGR):**
  - 2019: 15% (0.15)
  - 2020: 8% (0.08)
  - 2021: 12% (0.12)

- **Patent data:**
  - 2019: 3 patents
  - 2020: 5 patents
  - 2021: 2 patents

1. **Calculate 3-year rolling average AGR:**

$$
\mathrm{AGR}_{\text{3yr-avg}} = \frac{0.15 + 0.08 + 0.12}{3} = \frac{0.35}{3} = 0.117 \text{= 11.7 prct.}
$$

2. **Calculate 3-year cumulative patents:**

$$
\mathrm{Patents}_{\text{3yr-sum}} = 3 + 5 + 2 = 10  \text{patents}
$$

3. **Calculate Tech Gap (raw):**

$$
\mathrm{TechGap}_{\text{raw}} = \frac{\mathrm{AGR}_{\text{3yr-avg}}}{\mathrm{Patents}_{\text{3yr-sum}}} = \frac{0.117}{10} = 0.0117
$$

4. **Min–Max normalization over the entire sample (all companies, all rolling windows):**  
Assumed global minimum = −0.404  
Assumed global maximum = 0.724

$$
\mathrm{TechGap}_{\text{minmax}} = \frac{0.0117 - (-0.404)}{0.724 - (-0.404)} = \frac{0.0117 + 0.404}{1.128} = \frac{0.4157}{1.128} = 0.369
$$



This Min–Max normalized Tech Gap value of 0.369 locates the company at approximately 36.9% between the worst and best observed performers, allowing fair comparison across the full portfolio.

#### Step 3: Correlation Analysis
- Computed Pearson and Spearman correlations between tech gap (current year) and revenue 
  growth (current and lagged periods)
- Applied 3-sigma outlier detection to flag anomalies
- Stratified analysis by value chain segment

#### Step 4: Time-Lag Testing
- Tested lagged correlations: does t-1 tech gap predict t revenue growth?
- Evaluated statistical significance (p < 0.05 threshold) across years
- Assessed decay of predictive signal at lags 0, 1, 2, 3 years

### Methodology Note

- The Tech Gap KPI balances the 3-year average financial momentum (AGR) against cumulative innovation activity (patents) to identify potentially disruptive SATCOM companies.
- Min–Max normalization uses global bounds computed over all companies and years to standardize values; a 3-year window aligns with data availability, though a 5-year window would ideally provide smoother metrics if data suffices.
- The analysis employed standard statistical methods: Pearson and Spearman correlations 
calculated year-by-year, with with lagged correlation analysis to test predictive power. Initial validation code was developed collaboratively; the implementation has been independently rewritten based on scipy documentation and statistical best practices to ensure reproducibility and full 
methodological ownership.

### Results

| Metric | Value | Interpretation |
|--------|-------|-----------------|
| Mean Pearson correlation (2014–2024) | 0.606 | Moderate relationship; suggests additional confounding factors |
| Peak correlation (2018) | 0.974 | Outlier year; likely driven by market event or single large firm |
| Lag-1 predictive signal | 0.559 | Prior-year tech gap predicts current-year growth; significant (p<0.05) in 4 of 11 years |
| Companies flagged (repeated disruption) | 24 | >1 year with above-median tech gap; potential strategic inflection points |

**Key Insight:** Patent intensity exhibits modest forward-looking predictive power: the average lag‑1 correlation is r=0.559, suggesting that innovation activity typically requires 1–2 years to materialize in revenue. Cross‑sectional same‑year correlations are slightly higher (r=0.606), consistent with a short lag structure.

**Statistical Significance:** Four of eleven years (2018–2020, 2023) showed statistically 
significant correlations (p<0.05), indicating the relationship is real and not driven by 
random chance.

### Limitations & Future Work

1. **Data bias:** Patent databases exclude non-disclosure agreements and defensive patenting; 
   actual R&D intensity may differ.
2. **Survivor bias:** Bankruptcies/acquisitions reduce sample size in later years.
3. **Time bias:** Correlating AGR with nearby years (t and t±1) can inflate associations due to overlapping rolling windows and macro cycles; a stricter out‑of‑sample design (e.g., walk‑forward validation) and non‑overlapping windows are needed for deployment‑grade claims.
4. **Confounding variables:** Market shifts, regulatory changes, M&A not accounted for.
5. **Sample size:** 95 firms is modest; stratified analysis by segment (n=30–40) increases noise.

Recommended extensions: integrate R&D spending, control for industry 
cyclicality, test nonlinear relationships (neural networks, GAMs).

---
## Installation & Usage
```bash
# Download repository
git clone git@github.com:404rorre/patent-signal-analytics.git
cd patent-signal-analytics
# Create virtual environment 
python -m venv .venv
source .venv/bin/activate
pip install -r requirements
# Open Script
jupyter notebook technology_gap.ipynb
```
---

## License

MIT License — See LICENSE file for details
