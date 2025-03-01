# Quant Finance Modelling

## Task Overview

### What You'll Learn
- Understand valuing fintech loan portfolios.
- Analyze historical data for repayment percentages.
- Forecast cash flows and portfolio value.

### What You'll Do
- Create a valuation document.
- Analyze historical data and compute repayment rates.
- Forecast cash flows and determine the portfolio's present value.

## Background

The demand for credit has evolved beyond traditional bank loans, driven by e-commerce growth. Fintech companies have introduced innovative credit solutions, such as flexible payment schedules and buy-now-pay-later models. These new credit instruments require novel valuation approaches, particularly for merchant loans, where repayments depend on sales volume rather than fixed schedules.

Our client, a global online lending platform, provides loans to consumers and merchants. These loans are classified as assets on the balance sheet. Our audit colleagues have requested our expertise to verify the balance sheet values, ensuring the loan portfolio valuation is accurate.

## Objective

As part of the quantitative finance team, you will:
- Inspect historical data (June 2019 - December 2020) and compute repayment percentages.
- Compute expected repayment percentages over the loan lifetime.
- Forecast cash flows based on origination amounts.
- Compute the portfolio's present value using a given discount rate.
- Compare your valuation with the client's estimate (CHF 84,993,122.67) and determine if the difference is within the acceptable threshold of CHF 500,000.

## Steps for Analysis

1. **Inspect Historical Data**
   - Data includes loan origination amounts and observed repayments for each vintage.
   - Monthly repayment data is available up to December 2020.

2. **Compute Historical Repayment Percentages**
   - Compute repayment percentages as the share of origination amounts.
   - Clip values to ensure they remain within [0,1].

3. **Forecast Expected Repayments**
   - Use historical data to estimate future repayments.
   - Normalize repayment forecasts to ensure total repayments do not exceed 100%.

4. **Compute Forecasted Cash Flows**
   - Multiply expected repayment percentages by origination amounts.

5. **Discount Future Cash Flows**
   - Use an annual discount rate of 2.5% converted to a monthly rate.
   - Discount all cash flows back to December 31, 2020.

6. **Compute Portfolio Present Value**
   - Sum the discounted cash flows.
   - Compare with the client’s estimate.

7. **Assess Acceptability of Results**
   - Compute absolute and relative differences.
   - Determine if the difference is within the CHF 500,000 threshold.

## Code Implementation (Python)

```python
import pandas as pd
import numpy as np

# Load data
df = pd.read_csv('/content/Data.csv', sep=';')

df.rename(columns={'Unnamed: 0': 'Vintage Date'}, inplace=True)
df['Vintage Date'] = pd.to_datetime(df['Vintage Date'], format='%d.%m.%Y')

# Compute Historical Repayment Percentages
repayment_columns = df.columns[2:]
df_rep_percent = df.copy()
df_rep_percent[repayment_columns] = df[repayment_columns].div(df['Origination Amount'], axis=0)
df_rep_percent[repayment_columns] = df_rep_percent[repayment_columns].clip(0, 1)

# Forecast Repayments
def forecast_repayments(row):
    forecasted = [row.iloc[1]]
    if pd.isna(row.iloc[2]):
        forecasted.append(2 * row.iloc[1])
    else:
        forecasted.append(row.iloc[2])
    for i in range(3, 31):
        prev_sum = sum(forecasted)
        log_input = max(1e-3, 1 + (1 - (i - 1) / 30) * (1 - prev_sum))
        p_i = max(forecasted[1] * np.log(log_input), 0)
        forecasted.append(p_i)
    total_repayment = sum(forecasted)
    if total_repayment > 1.0:
        forecasted = [p / total_repayment for p in forecasted]
    return forecasted

df_forecast = df_rep_percent.apply(forecast_repayments, axis=1, result_type='expand')

# Compute Forecasted Cash Flows
df_cash_flows = df_forecast.multiply(df['Origination Amount'], axis=0)

# Discounting
annual_rate = 0.025
monthly_rate = (1 + annual_rate) ** (1/12) - 1
discount_factors = [(1 / (1 + monthly_rate)) ** i for i in range(1, 31)]
df_discounted = df_cash_flows.multiply(discount_factors, axis=1)

# Compute Present Value
portfolio_value = df_discounted.sum().sum()

# Compare with Client's Estimate
client_estimate = 84993122.67
difference = abs(portfolio_value - client_estimate)
relative_difference = (difference / client_estimate) * 100

print(f"Computed Portfolio Value: CHF {portfolio_value:,.2f}")
print(f"Absolute Difference: CHF {difference:,.2f}")
print(f"Relative Difference: {relative_difference:.4f}%")
print("Acceptable Difference?", "Yes" if difference < 500000 else "No")
```

### Sample Output
```
Computed Portfolio Value: CHF 397,969,594.42
Absolute Difference: CHF 312,976,471.75
Relative Difference: 368.2374%
Acceptable Difference? No
```

## Conclusion

The computed portfolio value significantly deviates from the client’s estimate. Further investigation is needed to:
- Verify the correctness of repayment forecasting methodology.
- Adjust discounting assumptions if necessary.
- Cross-check historical repayment data with client records.

This project provides a structured approach to fintech loan valuation and highlights the challenges of forecasting stochastic cash flows accurately.

