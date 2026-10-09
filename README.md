# Loan Disbursement Analysis | Power BI

An interactive Power BI dashboard that tracks loan disbursements, loan officer performance, and portfolio risk for a microfinance-style lending portfolio. It answers three business questions:

1. How much did we disburse each month over the last 6 months?
2. Which loan officers disbursed the most loans (and how much volume) last month?
3. What share of the outstanding portfolio is at risk (overdue more than 30 days)?

> **Screenshot:** add your dashboard image here, e.g. `![Dashboard](images/dashboard.png)`

---

## Business Requirements

| # | Requirement | Visual used |
|---|---|---|
| 1 | Monthly disbursements in EUR for the last 6 months (`principal_amount_eur`) | Column + line combo chart |
| 2 | Loans disbursed per loan officer for the last month: count and `sum(principal_amount_eur)` | Top 10 bar chart + detail table |
| 3 | Portfolio at Risk (PAR 30): outstanding principal with `overdue_days > 30` divided by total outstanding principal | KPI visual |
| 4 | Filters by `branch_key`, `product`, and `loan_officer` | Three slicers |

---

## Dataset

| File | Rows | Description |
|---|---|---|
| `Loan_Details.xlsx` | 26,560 | One row per loan: branch, product, loan officer, disbursement date, principal in EUR |
| `Loans.xlsx` | 20,931 | One row per loan: outstanding principal in EUR, overdue days, loan status |

Both tables are joined on `loan_key` (one-to-one).

**Data period:** loans disbursed from Jan 2014 to 23 Jul 2020.

---

## Data Preparation (Power Query)

- Kept only the columns needed for the analysis.
- Converted `disbursment_dateid` (a number such as `20200723`) into a real **Disbursement Date**:
  ```
  #date(Number.IntegerDivide([disbursment_dateid],10000),
        Number.Mod(Number.IntegerDivide([disbursment_dateid],100),100),
        Number.Mod([disbursment_dateid],100))
  ```
- Changed `branch_key` to text so it is not summed.
- Replaced 1,041 blank `product_key` values with `Unknown`.

---

## Data Model

```
Calendar (Date) 1 ───► * LoanDetails (Disbursement Date)
LoanDetails (loan_key) 1 ◄──► 1 Loans (loan_key)    [cross-filter: Both]
```

The one-to-one relationship uses **both-direction** filtering, so slicers on branch, product, and loan officer also filter the overdue and outstanding figures.

### Calendar helper columns

`Months Ago` makes every "last N months" filter relative to the **latest date in the data** rather than today's date. This keeps the report correct even when the data is not fully up to date.

```DAX
Months Ago =
VAR LatestDate = MAX(LoanDetails[Disbursement Date])
RETURN
    (YEAR(LatestDate) * 12 + MONTH(LatestDate))
  - (YEAR('Calendar'[Date]) * 12 + MONTH('Calendar'[Date]))
```

`0` = latest month (Jul 2020), `1` = Jun 2020, and so on. The 6-month chart filters `Months Ago` between 0 and 5, and the officer visuals filter `Months Ago = 0`.

---

## Key DAX Measures

```DAX
Total Disbursed EUR = SUM(LoanDetails[principal_amount_eur])

Loans Disbursed = DISTINCTCOUNT(LoanDetails[loan_key])

Total Outstanding EUR = SUM(Loans[principal_amount_outstanding_eur])

Outstanding Overdue 30+ EUR =
CALCULATE([Total Outstanding EUR], Loans[overdue_days] > 30)

PAR 30 % = DIVIDE([Outstanding Overdue 30+ EUR], [Total Outstanding EUR])

PAR 30 % (Overall) = CALCULATE([PAR 30 %], ALL('Calendar'))

Disbursed Last Month EUR =
CALCULATE([Total Disbursed EUR], 'Calendar'[Months Ago] = 0)

Loans Disbursed Last Month =
CALCULATE([Loans Disbursed], 'Calendar'[Months Ago] = 0)
```

---

## Dashboard Contents

- **Slicers:** Branch, Product, Loan Officer
- **Summary cards:** Total Outstanding EUR, Outstanding Overdue 30+ EUR, Disbursed Last Month EUR, Loans Disbursed Last Month
- **KPI:** Portfolio at Risk (30+ days) vs. a 5% placeholder target
- **Combo chart:** monthly disbursement volume (columns) and loan count (line), last 6 months
- **Bar chart:** Top 10 loan officers by loans disbursed last month
- **Table:** all loan officers with loan count and disbursed volume (data bars)

---

## Key Findings

| Month | Disbursed (EUR) | Loans |
|---|---|---|
| Feb 2020 | 768,397 | 439 |
| Mar 2020 | 249,139 | 181 |
| Apr 2020 | 4,334 | 6 |
| May 2020 | 24,892 | 30 |
| Jun 2020 | 728,847 | 377 |
| Jul 2020 | 695,385 | 308 |

- **Last 6 months total:** about EUR 2.47M disbursed.
- **Sharp drop in Apr and May 2020**, followed by a strong recovery in June.
- **Last month (Jul 2020):** 308 loans worth about EUR 695K, disbursed by 63 loan officers. The top officer (JOUM) disbursed 21 loans.
- **Portfolio at Risk (30+ days) is about 90.8%.** This is driven by written-off loans, which hold roughly EUR 3.87M of the EUR 4.48M total outstanding balance. Looking only at loans with status `OUTSTANDING`, PAR 30 is about 14.7%.

---

## Data Quality Notes

- 730 loans in `Loans.xlsx` have no match in `Loan_Details.xlsx`, so they show as (Blank) in the slicers. Blanks are removed from slicer lists, but these loans still count in portfolio totals when no filter is selected.
- Two loans disbursed on 30 Dec 2016 show EUR 125M and EUR 120M, where the EUR value equals the USD amount. This looks like a missing currency conversion. They fall outside the 6-month window, so the requested visuals are unaffected, but they would distort any all-time view.
- "Last month" is the latest month in the data (Jul 2020, partial up to the 23rd). To use the last full month instead, change the `Months Ago` filter from `0` to `1` (and `0-5` to `1-6` for the 6-month chart).

---

## How to Use

1. Clone or download this repository.
2. Open `Loan_Disbursement_Analysis.pbix` in Power BI Desktop.
3. If prompted, update the data source paths: **Home → Transform data → Data source settings**, and point to the two Excel files.
4. Click **Refresh**.

---

## Skills Demonstrated

Power Query data cleaning | Data modelling (one-to-one and one-to-many relationships) | Date table design | DAX (CALCULATE, DIVIDE, time intelligence logic) | KPI design | Slicers and interactivity | Dashboard layout and formatting

---

## Tools

Power BI Desktop | Power Query | DAX | Excel

---

## Author

**[Your Name]**
[LinkedIn](https://www.linkedin.com/in/your-profile) | [GitHub](https://github.com/your-username)

*Project inspired by the "Analytics with Nags" Power BI project series.*
