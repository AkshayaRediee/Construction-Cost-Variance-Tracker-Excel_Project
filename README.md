# 🏗️ Construction Cost Variance Tracker
### Dynamic Monthly Budget vs Actual Reporting Model — Excel

![Excel](https://img.shields.io/badge/Tool-Microsoft%20Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=flat)
![Domain](https://img.shields.io/badge/Domain-Construction%20Finance-0078D4?style=flat)
![Type](https://img.shields.io/badge/Type-Financial%20Model-8B5CF6?style=flat)

---

## 📌 What Is This Project?

This is a dynamic financial reporting model built entirely in Microsoft Excel. It takes a raw construction cost ledger hundreds of messy invoice line items and turns it into a clean, professional one-page executive summary that automatically updates every time you change a single dropdown.

Think of it like a live dashboard for a construction project. The project director walks into a client meeting, picks the reporting month from a dropdown, and instantly sees exactly where every dollar has gone, which trade packages are over budget, and what the project is forecast to cost at completion.

No manual copy-pasting. No rebuilding charts. No formula errors. Change one cell the whole report updates.

---

## 🎯 The Business Problem This Solves

Every junior analyst at firms in Cost&Construction Management does this manually every single month-end:

1. Export raw data from the project accounting system
2. Copy numbers into a template
3. Recalculate variances by hand
4. Reformat the table
5. Rebuild the chart

That process takes 3 – 4 hours every month. One mislinked cell or a formula error means the project director walks into a client meeting with wrong numbers in front of a client spending $200 million on a build.

This model eliminates that entirely. Paste new data, refresh, done in 10 minutes.

---

## 🏢 Real-World Context

| Role | How They Use This Model |
|---|---|
| **Project Director** | Uses the Executive Summary in client meetings to show exactly where money has gone and where it's heading |
| **Cost Manager** | Uses the RAG flags to have proactive conversations with contractors before an overrun becomes a crisis |
| **Client** | Uses the forecast column to decide whether to approve additional funding or push back on a contractor |

---

## 📁 Project Structure
Construction_Cost_Tracker

├── Construction_Cost_Tracker_Datasets.xlsx   ← Raw synthetic datasets (input)

├── Construction_Cost_Tracker.xlsx            ← The working model (output)

└── README.md                                 ← You are here

---

## 🗂️ Workbook Sheet Structure

| Sheet | Colour | Purpose |
|---|---|---|
| RAW_DATA | Grey | Raw invoice ledger 686 line items, every invoice for every trade |
| BUDGET_REGISTER | Blue | Approved budgets per trade and cost code, including change orders |
| MONTHLY_SUMMARY | Orange | Month-by-month planned vs actual spend per trade |
| EXECUTIVE_SUMMARY | Green | The one-page report this is what clients see |
| SCURVE_CHART | Purple | S-Curve chart data and visualisation |

---

## 📊 Dataset

The synthetic dataset replicates a real NYC capital infrastructure project. It covers a 12-month construction programme (Jan-2024 to Dec-2024) across 6 trade packages with a total project value of approximately $159.6 million.

### Trade Packages and Budgets

| Trade Package | Approved Budget |
|---|---|
| Civil | $19,012,000 |
| Structural Steel | $33,301,000 |
| MEP (Mechanical, Electrical, Plumbing) | $42,925,000 |
| Concrete | $22,972,000 |
| Fit-Out | $29,875,000 |
| Preliminaries | $11,560,000 |
| **TOTAL** | **$159,645,000** |

---
## 🔧 Formulas Explained

### The Month Dropdown

A Data Validation dropdown is placed in the Reporting Month cell. 
It contains the 12 months of the project (Jan-24 through Dec-24).

This single cell controls the entire model. Every formula references 
this one cell. Change the month — the whole report updates.

A helper cell converts the selected month into a number:
```excel
=MONTH(reporting_month_cell)
```
January = 1, February = 2, September = 9 and so on.
This number is used in formulas to filter invoices up to
and including the selected month.

---

### Approved Budget

```excel
=SUMIFS(
  Budget_Table[Current_Budget],
  Budget_Table[Trade],
  TRIM(trade_name_cell)
)
```

Goes into the budget table and adds up every budget line 
that belongs to the matching trade package.

TRIM is wrapped around the trade name to remove any invisible 
spaces that would cause the lookup to fail silently.

---

### Actual to Date

```excel
=SUMPRODUCT(
  (TRIM(Ledger_Table[Trade]) = TRIM(trade_name_cell)) *
  (Ledger_Table[Status] = "Posted") *
  (MONTH(Ledger_Table[Month]) <= month_number_helper_cell) *
  (Ledger_Table[Actual_Cost])
)
```

This is the most important formula in the model. It scans every 
invoice in the ledger and only includes it if three conditions 
are ALL true:

1. The invoice belongs to the correct trade
2. The invoice has been posted (it is a real invoice, not a forecast)
3. The invoice month is on or before the selected reporting month

The asterisks between conditions work like AND logic — all three 
must be TRUE for that invoice to be included in the total.

Why SUMPRODUCT instead of SUMIFS?
SUMIFS can only match exact values. SUMPRODUCT can handle 
comparisons like "less than or equal to" which is what we need 
to accumulate spend up to a selected month dynamically.

---

### Variance in Dollars

```excel
= Actual_to_Date - Approved_Budget
```

Simple subtraction. 
Negative number = under budget (money still available).
Positive number = over budget (overspending).

---

### Variance Percentage

```excel
= IFERROR(Variance_Dollar / Approved_Budget, 0)
```

Expresses the variance as a percentage of the approved budget.
IFERROR handles the case where budget is zero, preventing 
a divide-by-zero error from breaking the report.

---

### Percentage Complete

```excel
= IFERROR(Actual_to_Date / Approved_Budget, 0)
```

How much of the budget has been spent so far.
In construction, spend percentage is a reliable proxy 
for physical progress on site.

---

### Forecast Final Cost

```excel
= IFERROR(
    Approved_Budget + (Actual_to_Date - Approved_Budget) * (1 - Pct_Complete),
    Approved_Budget
  )
```

Answers the question: if we keep spending at this rate, 
what will this trade cost when it is 100% complete?

It takes the current variance and projects it forward 
across the remaining work still to be done.
This is earned value logic — the same methodology used 
by professional cost consultancies worldwide.

---

### Contingency Used

```excel
=SUMIFS(
  Budget_Table[Contingency],
  Budget_Table[Trade],
  TRIM(trade_name_cell)
)
```

Pulls the contingency reserve allocated to each trade 
from the budget register.

Comparing contingency against the variance tells the cost 
manager whether an overrun is within the safety envelope 
or has exceeded it and needs escalating.

---

### RAG Status

```excel
=IF(Pct_Complete < 0.5, "RED",
   IF(Pct_Complete < 0.7, "AMBER",
   "GREEN"))
```

Automatic traffic light based on how far through 
each trade is by the reporting month.

| Status | Condition | Meaning |
|---|---|---|
| 🔴 RED | Under 50% complete | Seriously behind — urgent attention needed |
| 🟡 AMBER | 50–70% complete | Slightly behind — monitor closely |
| 🟢 GREEN | Over 70% complete | On track |

Conditional formatting automatically colours the cell 
red, amber, or green based on the text value — 
no manual formatting needed ever again.

---

### S-Curve Chart — Planned Cumulative

```excel
=SUMPRODUCT(
  (MONTH(Ledger_Table[Month]) <= current_row_month_number) *
  (Ledger_Table[Planned_Cost])
)
```

Adds up all planned costs across all trades up to 
and including each month, building the planned S-Curve line.

### S-Curve Chart — Actual Cumulative

```excel
=SUMPRODUCT(
  (Ledger_Table[Status] = "Posted") *
  (MONTH(Ledger_Table[Month]) <= current_row_month_number) *
  (Ledger_Table[Actual_Cost])
)
```
How to read it:
- Actual line below planned line = work is behind schedule
- Actual line above planned line = ahead of schedule or costs are higher than planned
- The gap between the two lines at the reporting date = current cumulative variance
- Actual line going flat after September = no more posted invoices, project is mid-construction

Same logic but only includes posted invoices.
This is why the actual line goes flat after September 
there are no posted invoices for future months yet.
That flat line represents the work still to be completed.


## 🛠️ Skills Demonstrated

| Skill | Application |
|---|---|
| Excel Tables and Structured References | Named tables with column-name references for readable, maintainable formulas |
| SUMIFS | Multi-condition aggregation filtering by trade, status, and month |
| SUMPRODUCT | Advanced aggregation with comparison operators for cumulative filtering |
| Data Validation Dropdown | Single-cell control driving the entire model |
| IFERROR | Error-proofing to prevent #DIV/0! and #VALUE! errors |
| MONTH function | Converting date values to numbers for comparison logic |
| Conditional Formatting | Automatic RAG colour-coding, data bars, and variance highlighting |
| Financial Modelling Standards | Currency formatting, accounting style, freeze panes, hidden helper cells |
| S-Curve Chart Design | Dual series line chart with professional axis formatting |

---

## 🚀 How to Use This Model

1. Download both Excel files from this repository
2. Open Construction_Cost_Tracker.xlsx
3. Go to the EXECUTIVE_SUMMARY sheet
4. Change the dropdown in cell B4 to your reporting month
5. The entire report updates automatically

To use with your own project data, replace the contents of RAW_DATA, BUDGET_REGISTER, and MONTHLY_SUMMARY with your actual data. Keep the column names and table names identical.

---

## 👤 About

Built as a portfolio project demonstrating financial modelling and data analysis skills applicable to cost consultancy, project controls, and construction finance roles.

Skills: Excel financial modelling · Construction cost management · Dynamic reporting · Variance analysis · Earned value concepts · Data visualisation

---

*Built from scratch, raw data, formula logic, report design, and documentation as a demonstration of end-to-end analytical thinking in a construction finance context.* 




