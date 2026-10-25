# Price Elasticity & Profit Optimization Analytics

## Project Overview

This project explores retail price elasticity and evaluates
hypothetical pricing scenarios using Python, statistical
regression, and sensitivity analysis.

The objective is to understand how observed price and quantity
patterns relate to demand and how alternative pricing assumptions
may affect revenue and profit indices.

## Dataset

Source:
https://www.kaggle.com/datasets/suddharshan/retail-price-optimization

The dataset contains retail product-month observations, including
product prices, quantities, competitor prices, freight prices,
and other product-related variables.

## Tools & Technologies

- Python
- Pandas and NumPy
- Matplotlib
- Statsmodels
- Jupyter Notebook
- Statistical regression
- Sensitivity analysis

## Analytical Workflow

1. Data inspection and cleaning
2. Product-level observation eligibility checks
3. Exploratory data analysis
4. Price elasticity regression
5. Time-trend specification comparison
6. HC3 and HAC robust inference
7. Leave-one-out sensitivity analysis
8. Revenue scenario analysis
9. Hypothetical cost sensitivity analysis
10. Business recommendations and limitations

## Case Study: watches1

- Observations: 17
- Estimated elasticity: -5.862
- HC3 p-value: 0.0003
- R-squared: 0.605
- Price-time correlation: -0.810
- Leave-one-out estimates retain a negative sign.

The estimated coefficient suggests a negative relationship
between price and quantity under the fitted model. However,
the sample is small and price is strongly correlated with time.
The coefficient should therefore be treated as exploratory,
not as a causally identified price elasticity.

## Scenario Analysis

The project evaluates hypothetical price changes and compares
revenue and profit indices under different assumed unit costs.

These scenarios rely on a constant-elasticity assumption.
Unit costs are illustrative assumptions, not verified business
data. Scenario results are not actual profit forecasts and do
not establish a proven optimal price.

## Key Limitations

- Small product-level sample sizes
- Observational data and potential confounding
- Strong price-time correlation in the selected case study
- No verified unit-cost data
- Potential unobserved effects from promotions, inventory,
  seasonality, shipping, and competitor activity
- Scenario analysis does not establish causal price effects
- No controlled pricing experiment has been performed

## Recommended Next Steps

1. Obtain verified product costs and operating expenses.
2. Investigate promotions, inventory, and competitor pricing.
3. Collect more observations across products and time.
4. Evaluate suitable panel-data specifications where justified.
5. Run a controlled pricing experiment when feasible.
6. Validate any business decision with actual contribution margin.

## Reproducibility

1. Download the dataset from the Kaggle source above.
2. Place the CSV in the project's local data directory.
3. Open the analysis notebook in Jupyter.
4. Install the required Python libraries.
5. Run the notebook cells in sequence.

See `requirements.txt` for the core dependencies.

## Disclaimer

This is an exploratory analytics project for educational and
portfolio purposes. The findings should not be used directly
to set commercial prices without additional validation.
