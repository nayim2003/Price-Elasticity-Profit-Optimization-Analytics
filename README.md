# Price Elasticity & Profit Optimization Analytics

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Status](https://img.shields.io/badge/Status-Exploratory%20Analysis-orange)
![Method](https://img.shields.io/badge/Method-Log--Log%20Regression-green)

An end-to-end retail pricing analysis that audits a product-month sales dataset, estimates **price elasticity of demand** per product, tests how robust those estimates are, and translates them into **hypothetical revenue and profit scenarios** for a pricing experiment.

> **Read this first:** the results are *exploratory*. Sample sizes are small (12 to 19 observations per product), price is strongly correlated with time, no verified unit-cost data exists, and no controlled price experiment has been run. The scenarios show *what a pricing test is worth running*, not a proven optimal price.

---

## Table of Contents

1. [Business Problem](#1-business-problem)
2. [Key Results at a Glance](#2-key-results-at-a-glance)
3. [Dataset](#3-dataset)
4. [Project Structure](#4-project-structure)
5. [Methodology](#5-methodology)
6. [Detailed Findings](#6-detailed-findings)
7. [Scenario Analysis (Hypothetical)](#7-scenario-analysis-hypothetical)
8. [Business Recommendations](#8-business-recommendations)
9. [Limitations](#9-limitations)
10. [Data Quality Issues Found and Fixed](#10-data-quality-issues-found-and-fixed)
11. [How to Run](#11-how-to-run)
12. [Output Files](#12-output-files)
13. [Next Steps](#13-next-steps)
14. [Tech Stack](#14-tech-stack)
15. [Disclaimer](#15-disclaimer)

---

## 1. Business Problem

A retailer sells many products and wants to know:

- How strongly does demand react when a product's price changes?
- For which products is that reaction measurable with the available data?
- If prices were changed, what could happen to revenue and profit?
- What additional data or experiments are needed before changing prices?

**Price elasticity** (`e`) is the percentage change in quantity for a 1% change in price. If `e < -1`, demand is *elastic*: a price cut increases revenue. If `-1 < e < 0`, demand is *inelastic*: a price cut reduces revenue.

---

## 2. Key Results at a Glance

| Metric | Result |
|---|---|
| Records analysed | 676 product-month rows, 52 products, 9 categories |
| Time coverage | Jan 2017 to Aug 2018 (20 months) |
| Products eligible for elasticity modelling | **20 of 52** (>= 12 observations and >= 5 distinct prices) |
| Products whose elasticity sign flips when a time trend is added | **5 of 20** (unstable) |
| Products with robust, significant negative elasticity | **5** (`watches1`, `watches2`, `watches6`, `perfumery2`, `furniture2`) |
| Case-study product | `watches1` |
| `watches1` estimated elasticity | **-5.862** (HC3 p = 0.0003, R-squared = 0.605, n = 17) |
| `watches1` leave-one-out elasticity range | -6.917 to -5.274 (no sign flips) |
| `watches1` price-time correlation | -0.810 (confounding risk) |
| Cost data in dataset | **None** (profit scenarios use assumed costs) |

---

## 3. Dataset

**Source:** [Retail Price Optimization (Kaggle)](https://www.kaggle.com/datasets/suddharshan/retail-price-optimization)

Each row is **one product in one month** (676 rows, 30 columns, no missing values, no duplicate rows, no duplicate product-month pairs).

| Field group | Columns | Notes |
|---|---|---|
| Identifiers | `product_id`, `product_category_name`, `month_year` | `month_year` is **day-first (DD-MM-YYYY)** |
| Sales | `qty`, `unit_price`, `total_price`, `freight_price` | `total_price` is approximately `qty x unit_price` and **excludes freight** |
| Product attributes | `product_name_lenght`, `product_description_lenght`, `product_photos_qty`, `product_weight_g`, `product_score` | Original misspelling `lenght` is preserved |
| Demand context | `customers`, `weekday`, `weekend`, `holiday`, `month`, `year` | `customers` is strongly linked to quantity in the examined product |
| Competitors | `comp_1-3`, `ps1-3`, `fp1-3` | Competitor price, and (by naming) their score and freight price |
| Price history | `lag_price` | Previous month's price |

**Not in the dataset:** unit cost, COGS, margin or profit.

**Key statistics**

| Field | Min | Median | Mean | Max |
|---|---|---|---|---|
| `qty` | 1 | 10 | 14.5 | 122 |
| `unit_price` | 19.9 | 89.9 | 106.5 | 364 |
| `total_price` | 19.9 | 807.9 | 1,422.7 | 12,095 |

---

## 4. Project Structure

```text
price-elasticity-profit-optimization/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── raw/
│       └── retail_price.csv          # download from Kaggle (git-ignored)
├── notebooks/
│   └── price_elasticity_analysis.ipynb
└── outputs/
    ├── elasticity_results.csv
    ├── pricing_scenarios.csv
    ├── cost_sensitivity.csv
    └── business_recommendations.csv
```

---

## 5. Methodology

### 5.1 Workflow

```text
Raw data -> Quality audit -> Eligibility screening -> EDA
        -> Baseline log-log model -> Time-trend specification
        -> All-product comparison -> Robust inference (HC3, HAC)
        -> Leave-one-out sensitivity -> Revenue scenarios
        -> Profit scenarios (assumed cost) -> Cost sensitivity -> Recommendations
```

### 5.2 Model

A constant-elasticity (log-log) demand model is estimated **separately for each product**:

```text
Baseline:      ln(Q_t) = b0 + b1 * ln(P_t) + e_t
With trend:    ln(Q_t) = b0 + b1 * ln(P_t) + b2 * t + e_t
```

`b1` is the price elasticity. The time index `t` (one step per observed month) is added to separate the price effect from general growth, because the product catalogue and sales grew strongly over the period.

### 5.3 Why product-level, not pooled

The pooled price-quantity correlation across all 676 rows is only **-0.103**, because cheap and expensive products are mixed. Within products the picture is very different (and varies from -0.91 to +0.95), so elasticity is estimated per product.

### 5.4 Eligibility screening

A product is modelled only if it has **at least 12 monthly observations** and **at least 5 distinct prices**. This removes products with no price variation (for example `health1` and `health4`, which never change price). Result: **20 of 52 products**.

### 5.5 Inference and robustness

| Technique | Purpose |
|---|---|
| HC3 standard errors | Robust to heteroscedasticity in small samples |
| HAC (Newey-West, 1 lag) standard errors | Robust to autocorrelated residuals |
| Durbin-Watson statistic | Checks for residual autocorrelation |
| Price-only vs price + trend comparison | Detects confounding with time |
| Leave-one-out (LOO) re-estimation | Tests whether one observation drives the result |

### 5.6 Scenario formulas

With baseline price `P0` and elasticity `e`, for a price ratio `r = P / P0`:

```text
Quantity index   = r^e x 100
Revenue index    = r^(1 + e) x 100
Profit index     = (P - cost) x quantity index / (P0 - cost)
```

---

## 6. Detailed Findings

### 6.1 Data quality

- No missing values, no duplicate rows, no zero or negative quantities or prices.
- `total_price` matches `qty x unit_price` closely (mean absolute difference 1.23; 79 rows differ by more than 0.01). Adding freight makes the match much worse (21.18), so `total_price` is product revenue **excluding freight**.
- Product coverage grows from 2 products (Jan 2017) to 50 (Mar 2018), then falls to 38 (Aug 2018). Early months are thin, so growth over time partly reflects catalogue expansion.

### 6.2 Exploratory analysis

- Pooled price-quantity correlation: **-0.103** (misleading because of mixing products).
- Within-product correlations are mostly negative (for example `watches7` -0.77, `consoles1` -0.76, `watches8` -0.72) but several are **positive** (for example `health3` +0.95, `health6` +0.73, `garden2` +0.58). Positive values point to confounding such as promotions, seasonality or growth.
- In `computers4`, quantity is far more strongly linked to `customers` (+0.814) than to price (-0.314). In this product `comp_1` equals `unit_price` exactly, so it carries no independent competitor information.

### 6.3 The time-trend problem (`computers4`)

| Model | Elasticity | p-value | R-squared | AIC |
|---|---|---|---|---|
| Price only | -3.217 | 0.092 | 0.153 | 49.8 |
| Price + time trend | **+3.793** | 0.055 | 0.574 | 39.4 |

Adding a trend improves the fit but flips the sign to an implausible positive value. The log price and time index are correlated at **-0.795**: price fell while sales grew, so the two effects cannot be separated cleanly.

### 6.4 All 20 eligible products

- The elasticity **sign changes for 5 of 20 products** when the trend is added (`computers4`, `garden6`, `garden8`, `garden4`, `garden5`), so those estimates are unreliable.
- Some trend-adjusted estimates are economically implausible (`bed2` +8.65, `garden2` +14.75), which indicates confounding.
- Five products show a negative, statistically significant elasticity (p < 0.05) that survives the trend control.

### 6.5 Robustness of the five shortlisted products

| Product | n | Elasticity | HC3 p | HAC p | R-squared | Durbin-Watson | Price-time corr. | LOO range |
|---|---|---|---|---|---|---|---|---|
| `watches1` | 17 | **-5.862** | 0.0003 | 0.0000 | 0.605 | 2.22 | -0.81 | -6.917 to -5.274 |
| `watches2` | 15 | -6.497 | 0.0081 | 0.0013 | 0.425 | 2.16 | -0.75 | -7.491 to -5.075 |
| `watches6` | 14 | -10.800 | 0.0381 | 0.0067 | 0.330 | 1.99 | -0.89 | -13.430 to -8.166 |
| `perfumery2` | 13 | -8.554 | 0.0005 | 0.0000 | 0.781 | 2.42 | -0.72 | -9.818 to -7.378 |
| `furniture2` | 13 | -12.850 | 0.0008 | 0.0000 | 0.760 | 1.74 | +0.55 | -16.391 to -12.156 |

Observations:

- **No sign flips** in any leave-one-out run.
- `watches1` has the **narrowest LOO range** (1.64), so it was chosen as the case study.
- `furniture2` and `watches6` are the most sensitive (LOO ranges 4.23 and 5.26).
- `furniture2` is the only product where price and time are positively correlated, so its negative elasticity is less likely to be a pure trend artefact.
- Elasticities between -6 and -13 are **very large** for retail products. They should be read as a signal of strong price sensitivity in this sample, not as literal demand rules.

---

## 7. Scenario Analysis (Hypothetical)

Case study: `watches1`, baseline price = median observed price (**179.90**), elasticity **-5.862**.

### 7.1 Revenue index (baseline = 100)

| Price change | Scenario price | Quantity index | Revenue index |
|---|---|---|---|
| -20% | 143.92 | 369.9 | **295.9** |
| -15% | 152.92 | 259.3 | 220.4 |
| -10% | 161.91 | 185.5 | 166.9 |
| -5% | 170.90 | 135.1 | 128.3 |
| 0% | 179.90 | 100.0 | 100.0 |
| +5% | 188.90 | 75.1 | 78.9 |
| +10% | 197.89 | 57.2 | 62.9 |
| +15% | 206.88 | 44.1 | 50.7 |
| +20% | 215.88 | 34.3 | 41.2 |

Using the leave-one-out elasticity bounds, the revenue index at -20% ranges from about **259.5 to 374.5**. The direction is consistent, the magnitude is uncertain.

### 7.2 Profit index at an assumed unit cost of 100

| Price change | Unit margin | Profit index |
|---|---|---|
| -20% | 43.92 | **203.3** |
| -10% | 61.91 | 143.7 |
| -5% | 70.90 | 119.9 |
| 0% | 79.90 | 100.0 |
| +10% | 97.89 | 70.1 |
| +20% | 115.88 | 49.8 |

### 7.3 Sensitivity to the assumed unit cost (profit index)

| Price change | Cost 80 | Cost 100 | Cost 120 | Cost 140 |
|---|---|---|---|---|
| -20% | 236.7 | 203.3 | 147.7 | 36.3 |
| -10% | 152.1 | 143.7 | 129.8 | 101.8 |
| -5% | 122.9 | 119.9 | 114.8 | **104.6** |
| 0% | 100.0 | 100.0 | 100.0 | 100.0 |
| +10% | 67.5 | 70.1 | 74.4 | 83.0 |
| +20% | 46.7 | 49.8 | 55.0 | 65.3 |

| Assumed cost | Best tested price change | Profit index change |
|---|---|---|
| 80 | -20% | +136.7% |
| 100 | -20% | +103.3% |
| 120 | -20% | +47.7% |
| 140 | -5% | +4.6% |

**How to read this**

- Because demand is modelled as very elastic, lower prices raise revenue and (for low assumed costs) profit.
- The more costly the product, the less room for discounting: at a cost of 140 a 20% cut destroys most of the profit.
- For three of the four cost cases the best option sits on the **edge of the tested grid** (-20%), so the true optimum may lie beyond it. The notebook flags this and also flags whether each scenario price lies inside the observed price range (about 132.5 to 255.6 for `watches1`).
- All values are **model scenarios based on an assumed cost**, not forecasts.

---

## 8. Business Recommendations

| Area | Finding | Recommended decision |
|---|---|---|
| Where to act | Only 5 of 52 products have reasonably robust elasticity evidence | Limit any elasticity-based pricing work to `watches1`, `watches2`, `watches6`, `perfumery2`, `furniture2`. Handle the rest through experiments, not models |
| Pilot design | `watches1` is the most stable estimate and a cut is the direction suggested by the model | Run a controlled price test starting with a **modest cut (-5% to -10%)**, not -20%, and measure real volume and margin |
| Capacity | The model implies roughly 3.7x volume at -20% | Check stock, supply and fulfilment capacity before any large discount |
| Profit | No cost data exists | Obtain verified unit cost, shipping, discounts and returns before any profit claim |
| Confounding | Price and time move together (corr. -0.81) | Change prices independently of the calendar (A/B or randomised price points) and track promotions, seasonality and competitor prices |
| Reporting | Estimates are exploratory | Describe elasticities as *indicative* in any stakeholder communication |
| Weak products | `computers4` and others flip sign or are insignificant | Do not change prices of these products based on this analysis |

---

## 9. Limitations

1. **Small samples:** 12 to 19 monthly observations per modelled product.
2. **Confounding with time:** strong price-time correlation means price effects can be mixed with growth.
3. **Observational data:** a regression does not establish causality; there is no experiment.
4. **Unobserved drivers:** promotions, inventory, seasonality, shipping and competitor actions are not fully controlled.
5. **No cost data:** profit scenarios depend on assumed unit costs.
6. **Constant-elasticity assumption:** real elasticity usually changes with price level and distance from observed prices.
7. **Limited price grid:** the tested grid (-20% to +20%) does not prove a global optimum.
8. **Linear trend only:** a single linear time term may not capture seasonality or non-linear growth.
9. **Unequal time coverage:** catalogue growth means early months have far fewer products.

---

## 10. Data Quality Issues Found and Fixed

| Issue | Impact | Fix |
|---|---|---|
| Dates are **day-first** but were parsed month-first by default | Only 50 of 676 dates fell on day 1 and the date range was wrong | Explicit `format="%d-%m-%Y"` parsing |
| `total_price` vs `qty x unit_price` mismatch (79 rows) | Small (mean absolute difference 1.23) | Accepted as minor; elasticity uses `qty` and `unit_price` |
| `total_price` excludes freight | Revenue definition | `total_price` treated as product revenue; freight kept separate for profit |
| `comp_1` identical to `unit_price` in `computers4` | Not an independent competitor signal | Not used as a control |
| Hard-coded values in summary cells | Risk of stale numbers | Case-study values computed from fitted model objects |
| `requirements.txt` missed `seaborn` and `plotly` | Notebook failed in a fresh environment | Dependencies added |

---

## 11. How to Run

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd price-elasticity-profit-optimization

# 2. Create and activate a virtual environment
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS / Linux:
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download the dataset from Kaggle and place it at:
#    data/raw/retail_price.csv

# 5. Open the notebook and run all cells
jupyter notebook notebooks/price_elasticity_analysis.ipynb
```

> **Path note:** the notebook currently loads the CSV from an absolute local path. Change it to a relative path before running on another machine:
> ```python
> df = pd.read_csv("../data/raw/retail_price.csv")
> ```

Run cells **in order**: later scenario and summary cells reuse variables created by earlier cells.

---

## 12. Output Files

| File | Content |
|---|---|
| `outputs/elasticity_results.csv` | Validation summary for the `watches1` case study |
| `outputs/pricing_scenarios.csv` | Revenue index by price change |
| `outputs/cost_sensitivity.csv` | Profit index by price change and assumed unit cost |
| `outputs/business_recommendations.csv` | Business interpretation table |

The scenario and cost files contain **hypothetical** assumptions and must not be read as forecasts.

---

## 13. Next Steps

1. Obtain verified unit cost, freight, discount and returns data.
2. Add controls for promotions, holidays, seasonality, inventory and competitor prices.
3. Collect more observations (weekly rather than monthly data would help).
4. Evaluate panel-data models (product fixed effects) across products where justified.
5. Run a controlled pricing experiment on the shortlisted products.
6. Recalculate profit scenarios with actual contribution margin and extend the price grid only within the observed range.

---

## 14. Tech Stack

| Area | Tools |
|---|---|
| Language | Python |
| Data handling | pandas, NumPy |
| Visualisation | Matplotlib, Seaborn, Plotly |
| Statistics | statsmodels (OLS, HC3, HAC, Durbin-Watson) |
| Environment | Jupyter Notebook |

---

## 15. Disclaimer

This is an exploratory analytics project for educational and portfolio purposes. The findings should not be used to set commercial prices without additional validation, verified cost data and a controlled pricing experiment.
