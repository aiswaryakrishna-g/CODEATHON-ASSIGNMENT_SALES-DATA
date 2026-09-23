[README CODEATHON.md](https://github.com/user-attachments/files/32581906/README.CODEATHON.md)
<div align="center">

# 📊 Power BI Sales Data Analysis

### End-to-end data cleaning, DAX modeling & dashboard design in Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2ea44f?style=for-the-badge)

**Author:** Aiswarya Krishna G · **Dataset:** `SalesData_1000Rows_WithIssues_copy.csv`

</div>

---

## 📑 Table of Contents

- [Objective](#-objective)
- [Data Cleaning](#-data-cleaning-power-query)
- [Data Model](#-data-model)
- [DAX Formulas & Functions](#-dax-formulas--functions)
- [Dashboard Layout](#-dashboard-layout)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [Conclusion](#-conclusion)

---

## 🎯 Objective

Import, clean, and model a raw customer order dataset in Power BI, then build an interactive **three-page dashboard** revealing order and profit trends across regions, products, and time.

| Goal | Description |
|---|---|
| 🧹 **Clean** | Resolve missing values, duplicates, inconsistent dates, and invalid negative costs |
| 🏗️ **Model** | Build corrected calculated columns — `Sales_Fixed`, `Cost_Fixed`, `Profit_Fixed` |
| 🧮 **Calculate** | Create DAX calculated tables, columns, and measures |
| 📈 **Visualize** | Regional pie chart, top-products column chart, profit-trend line chart |
| 💡 **Recommend** | Turn insights into concrete sales actions for next year |

---

## 🧹 Data Cleaning (Power Query)

> The raw dataset had missing values, **18 fully duplicate order rows**, inconsistent date text, and invalid negative cost entries.

**Steps applied before loading the model:**

- ✅ Removed fully blank rows
- ✅ Sorted `OrderDate` ascending, filtered out blank dates
- ✅ Capitalized `CustomerName`; missing → `"Unknown Customer"`
- ✅ Missing `Region` → `"Unspecified"`, missing `Product` → `"Unspecified Product"`
- ✅ Fixed negative `Cost` values via **Transform → Number Column → Absolute Value**
- ✅ Removed 18 duplicates on `OrderID` (the natural key)
- ✅ Standardized `OrderDate` from text (`DD-MM-YYYY`) to a proper Date type

### 🔧 Reconstructing missing numeric values

`Sales`, `Cost`, and `Profit` are mathematically linked, so gaps were **rebuilt from related columns** instead of being dropped:

```m
// Sales_Fixed — rebuild Sales when missing or zero
Sales_Fixed =
if [Sales] = 0 or [Sales] = null
then [UnitPrice] * [Quantity]
else [Sales]

// Profit_Fixed — rebuild Profit from Sales and Cost
Profit_Fixed =
if [Profit] = 0 and [Sales_Fixed] <> [Cost]
then [Sales_Fixed] - [Cost]
else [Profit]

// Cost_Fixed — rebuild Cost from Sales and Profit
Cost_Fixed =
if [Cost] = 0
then [Sales_Fixed] - [Profit_Fixed]
else [Cost]
```

> 💡 `Sales_Fixed`, `Cost_Fixed`, and `Profit_Fixed` are the foundation for every DAX measure and visual that follows.

---

## 🗂️ Data Model

Single fact table: **`SalesData_1000Rows_WithIssues_copy`**

| Field | Type | Role |
|---|---|---|
| `OrderID` | 🔢 Whole Number | Unique order key (dedup) |
| `OrderDate` | 📅 Date | Time axis — Year/Quarter/Month hierarchy |
| `CustomerName` | 🔤 Text | Customer dimension (cleaned) |
| `Region` | 🔤 Text | Slicer + pie chart legend |
| `Product` / `Category` | 🔤 Text | Slicer + column chart axis |
| `Quantity`, `UnitPrice` | 🔢 Number | Basis for `Sales_Fixed` |
| `Sales_Fixed`, `Cost_Fixed`, `Profit_Fixed` | 🔢 Number (calc.) | Base for all DAX |

---

## 🧮 DAX Formulas & Functions

### 📋 Calculated Table — `EastRegionOrders`
```dax
EastRegionOrders =
FILTER(
    'SalesData_1000Rows_WithIssues_copy',
    'SalesData_1000Rows_WithIssues_copy'[Region] = "East"
)
```

### 📐 Calculated Column — `ProfitCostDifference`
```dax
ProfitCostDifference =
'SalesData_1000Rows_WithIssues_copy'[Profit_Fixed]
  - 'SalesData_1000Rows_WithIssues_copy'[Cost]
```
> ⚠️ Margin is only ~5.5%, so this aggregates to a **large negative number by design** — not an error.

### 📏 Calculated Measure — `TotalProfit`
```dax
TotalProfit = SUM('SalesData_1000Rows_WithIssues_copy'[Profit_Fixed])
```

### ➕ Supporting Measures
```dax
TotalSalesAmt   = SUM('SalesData_1000Rows_WithIssues_copy'[Sales_Fixed])
OrderCount      = COUNTROWS('SalesData_1000Rows_WithIssues_copy')
ProfitMarginPct = DIVIDE([TotalProfit], [TotalSalesAmt])
```

### 🛠️ Functions Reference

| Function | Category | Used For |
|---|---|---|
| `FILTER()` | DAX — Table | `EastRegionOrders` calculated table |
| `SUM()` | DAX — Aggregation | `TotalProfit`, `TotalSalesAmt` |
| `COUNTROWS()` | DAX — Aggregation | `OrderCount` |
| `DIVIDE()` | DAX — Math | Safe division for `ProfitMarginPct` |
| `MONTH()` / `YEAR()` | DAX — Date | Seasonal analysis columns |
| `FORMAT()` | DAX — Text/Date | `MonthName`, `MonthShort`, `YearText` |
| `IF()` | Power Query (M) | Rebuilding `Sales_Fixed`/`Profit_Fixed`/`Cost_Fixed` |
| `Table.Distinct()` | Power Query (M) | Removing 18 duplicate rows |
| `Table.ReplaceValue()` | Power Query (M) | Replacing nulls |
| `Table.TransformColumnTypes()` | Power Query (M) | `OrderDate` → Date (en-GB locale) |
| `Number.Abs()` | Power Query (M) | Fixing negative `Cost` values |

---

## 📊 Dashboard Layout

Three pages, one page navigator, synced slicers (**Region**, **Category**, **OrderDate**) across all pages:

<table>
<tr><td width="33%" valign="top">

### 1️⃣ Overview
- KPI cards: Sales, Profit, Orders, AOV
- 🥧 Regional pie chart
- 📊 Top-5 products column chart

</td><td width="33%" valign="top">

### 2️⃣ Trends
- 📈 Profit-over-time line chart
- Sales by Category bar chart
- `EastRegionOrders` table

</td><td width="33%" valign="top">

### 3️⃣ Insights & Recommendations
- Before/after cleaning stats
- Data quality issue → fix cards
- Findings below 👇

</td></tr>
</table>

---

## 💡 Key Insights

<table>
<tr><td>💰</td><td><b>Total profit</b></td><td>₹5.14M on the full dataset, at roughly a <b>5.5% margin</b>. Margin is thin and fairly flat across products (5.0%–6.3%), so profit growth needs to come from volume or margin improvement — no single product drags it down.</td></tr>
<tr><td>📅</td><td><b>Sales pattern</b></td><td>The reliable pattern is <b>seasonal, not yearly</b>. <b>Dec, Aug, Mar</b> are consistently strongest; <b>Jun, Sep, Nov</b> are consistently weak. Year-over-year "trend" isn't trustworthy since <code>OrderDate</code> is scattered unrealistically across 1970–2025 — but the monthly pattern held up even on filtered subsets.</td></tr>
<tr><td>🏆</td><td><b>Product performance</b></td><td><b>Monitor</b> sells the most (420 units, ₹12.9M); <b>Smartphone</b> sells the least on every measure (324 units, ₹9.8M, lowest profit too). Laptop & Bookshelf have the best margins (6.0–6.3%).</td></tr>
<tr><td>🌍</td><td><b>Region</b></td><td><b>East</b> leads (₹25.0M), <b>West</b> trails (₹20.7M) — about a 17% gap.</td></tr>
</table>

---

## 🚀 Recommendations

**How to improve sales next year:**

1. 🎁 **Push promotions and stock** ahead of Dec/Aug/Mar, and run targeted discounts in Jun/Sep/Nov to flatten the slow periods.
2. 📱 **Revive Smartphone** specifically — bundle it with Headphones, review its pricing, since it underperforms on both volume and profit.
3. 💻 **Shift merchandising** toward Laptop and Bookshelf (better margins) rather than just chasing Monitor/Cabinet volume.
4. 🧭 **Study what's working in East** and apply it to West.
5. 🛠️ **Fix the `OrderDate` data quality issue** so next year's actual trend can be measured properly — right now you can't reliably tell if sales are growing or shrinking year over year.

---

## ✅ Conclusion

This project demonstrates a complete Power BI workflow: importing a messy real-world dataset, applying systematic data cleaning in Power Query (reconstructing missing numeric values from related columns rather than discarding them), building calculated tables, columns, and measures in DAX, and designing a three-page interactive dashboard.

Beyond the technical build, the analysis surfaced an important data quality finding — that apparent year-over-year trends in this dataset are **not reliable** — while still extracting genuine, actionable insights on seasonality, product performance, and regional differences to inform sales strategy for the upcoming year.

<div align="center">

---

Made with 📊 in Power BI

</div>
