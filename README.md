# Credit_Card_Analysis_Report

# Credit Card Weekly Dashboard (Power BI)

A weekly credit card dashboard that tracks key performance metrics and trends, so stakeholders can monitor and analyze credit card operations.

## Tools
- **Power BI** – dashboard and DAX measures
- **PostgreSQL** – source tables (`cc_detail`, `cust_detail`)

## Dashboard Pages
1. **Credit Card Transaction Report** – Revenue, Interest, Amount, Count, quarterly trend, revenue by card type / expenditure type / customer job, delinquency rate.
2. **Credit Card Customer Report** – Revenue by income group, age group, gender, state, marital status, dependents, customer job.
3. **Weekly WoW Insights** – Week-over-week change in revenue, transaction amount and transaction count, plus activation and delinquency rates.

Slicers: quarter, week start date, gender, income group, card category.

## DAX Measures

```dax
Revenue = 'public cc_detail'[annual_fees]
        + 'public cc_detail'[total_trans_amt]
        + 'public cc_detail'[interest_earned]

weeknum2 = WEEKNUM('public cc_detail'[week_start_date])
-- used to sort weeks in order

Current_week_Revenue = CALCULATE(
    SUM('public cc_detail'[Revenue]),
    FILTER(ALL('public cc_detail'),
        'public cc_detail'[weeknum2] = MAX('public cc_detail'[weeknum2]))
)

Pervious_week_Revenue = CALCULATE(
    SUM('public cc_detail'[Revenue]),
    FILTER(ALL('public cc_detail'),
        'public cc_detail'[weeknum2] = MAX('public cc_detail'[weeknum2]) - 1)
)

wow_growth = DIVIDE([Current_week_Revenue] - [Pervious_week_Revenue],
                    [Pervious_week_Revenue])
```

The same current/previous/growth pattern is used for **transaction amount** and **transaction count**.

## How Key Numbers Are Calculated

| Metric | Formula | Result |
|---|---|---|
| Revenue | annual fees + transaction amount + interest earned | 56,517,011 (~57M) |
| WoW revenue growth (Week 53) | (1,201,601 − 933,134) / 933,134 | 28.8% |
| WoW transaction amount growth | (1,011,008 − 748,677) / 748,677 | 35.0% |
| WoW transaction count growth | (11,583 − 11,203) / 11,203 | 3.4% |
| Blue + Silver share of transaction amount | (37,840,749 + 4,647,596) / 45,533,021 | ~93% |
| Delinquent rate | delinquent accounts / total accounts | 6.06% |
| Activation rate | accounts activated within 30 days / total accounts | 57.46% |

## Key Insights (YTD, Week 53)
- Revenue 57M, interest 8M, transaction amount 46M, transaction count 667K.
- Male customers contribute 31M revenue vs female 26M.
- Blue and Silver cards drive ~93% of transactions.
- TX, NY and CA together contribute ~68%.
- Week 53 revenue rose 28.8% WoW, after a 12.8% drop in Week 52.

## Data Refresh (Near Real-Time)
- The dashboard first loaded data from PostgreSQL up to **Week 52**.
- Week 53 data was then uploaded to PostgreSQL and the Power BI report was **refreshed**.
- The DAX uses `MAX(weeknum2)`, so the dashboard automatically treats the newest week (Week 53) as the current week and Week 52 as the previous week. No measure changes were needed.

## Weekly Update Process
1. Upload the new week's data to PostgreSQL.
2. Refresh the Power BI report.
3. Check that the WoW tables now show the new week as current.
4. Update the weekly insights document.
