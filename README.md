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

### The Month Dropdown — Cell B4

   Data Validation → List → Jan-24, Feb-24 ... Dec-24

  This single cell controls the entire model. Every formula references $B$4. Change this one cell and all 54 formula cells recalculate simultaneously.

Helper cell:
```excel
=MONTH(B4)
```
Converts the selected month into a number. January = 1, September = 9. Used in SUMPRODUCT to filter invoices up to the selected month.

---

### Approved Budget — Column B

```excel
=SUMIFS(table_budget[Current_Budget], table_budget[Trade], TRIM(A10))
```

Goes to the budget table, finds all rows matching the trade name, and adds up the Current Budget. TRIM removes invisible spaces that would cause a mismatch error.

---

### Actual to Date — Column C

```excel
=SUMPRODUCT(
  (TRIM(table_ledger[Trade]) = TRIM(A10)) *
  (table_ledger[Status] = "Posted") *
  (MONTH(table_ledger[Month]) <= $L$4) *
  (table_ledger[Actual_Cost])
)
```

This is the most important formula in the model. It filters every invoice by three conditions correct trade, posted status, and month up to the selected month and sums only the invoices that pass all three. The asterisks work like AND logic. This is why the dropdown updates the actuals dynamically.

SUMPRODUCT is used instead of SUMIFS because SUMIFS cannot handle less-than-or-equal comparisons for cumulative filtering.

---

### Variance in Dollars — Column D

```excel
=C10 - B10
```

Actual minus budget. Negative = under budget. Positive = over budget.

---

### Variance Percentage — Column E

```excel
=IFERROR(D10 / B10, 0)
```

Variance as a percentage of approved budget. IFERROR prevents a divide-by-zero error if budget is blank.

---

### % Complete — Column H

```excel
=IFERROR(C10 / B10, 0)
```

How much of the approved budget has been spent. In construction, spend percentage is a reliable proxy for physical progress.

---

### Forecast Final Cost — Column F

```excel
=IFERROR(B10 + (C10 - B10) * (1 - H10), B10)
```

Based on current spend rate, what will this trade cost at 100% completion? Takes the current variance and projects it forward across the remaining work. This is earned value logic the same methodology used by professional cost consultancies worldwide.

---

### Contingency Used — Column G

```excel
=SUMIFS(table_budget[Contingency], table_budget[Trade], TRIM(A10))
```

How much contingency reserve was allocated to this trade. Shows whether an overrun is within the contingency envelope or exceeding it.

---

### RAG Status — Column I

```excel
=IF(H10 < 0.5, "RED", IF(H10 < 0.7, "AMBER", "GREEN"))
```

Automatic traffic light status based on percentage complete.

| Status | Condition | Meaning |
|---|---|---|
| 🔴 RED | Under 50% complete | Seriously behind — urgent attention needed |
| 🟡 AMBER | 50–70% complete | Slightly behind — monitor closely |
| 🟢 GREEN | Over 70% complete | On track |

Conditional formatting automatically colours the cells based on the text value no manual formatting needed.

---

### S-Curve Chart Formulas

Total Planned Cumulative:
```excel
=SUMPRODUCT((MONTH(table_ledger[Month]) <= A2) * (table_ledger[Planned_Cost]))
```

Total Actual Cumulative:
```excel
=SUMPRODUCT((table_ledger[Status]="Posted") * (MONTH(table_ledger[Month]) <= A2) * (table_ledger[Actual_Cost]))
```

These formulas build the S-Curve data table row by row across all 12 months, giving the full curve shape regardless of the dropdown selection.

---

## 📈 The S-Curve Chart

The S-Curve is the signature visualisation of construction project management. It plots cumulative planned spend vs cumulative actual spend over the life of the project.

How to read it:
- Actual line below planned line = work is behind schedule
- Actual line above planned line = ahead of schedule or costs are higher than planned
- The gap between the two lines at the reporting date = current cumulative variance
- Actual line going flat after September = no more posted invoices, project is mid-construction

---

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




