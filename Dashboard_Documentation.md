# Dashboard Documentation – Accounts Payable KPIs Dashboard (Q1 2025)

This document provides the **technical overview and build documentation** for the *Accounts Payable KPIs Dashboard – Q1 2025 Overview* Power BI report.  
It covers the data source, model design, DAX measures, and visualization logic used to deliver actionable insights into payment efficiency and processing performance.

---

## 🧭 1. Project Overview

The **Accounts Payable KPIs Dashboard** provides a consolidated, data-driven view of Q1 2025 payment operations.  
It enables finance teams to track:

- **Payment efficiency** (average time to settle invoices)  
- **On-time vs. late payment performance**  
- **Rejection and reprocessing trends**  

The goal is to improve supplier relationships and operational cash flow visibility.

---

## 🧾 2. Data Sources

| Source | Description | Format | Refresh Frequency |
|--------|--------------|---------|-------------------|
| `Accounts_Payable_Sample.csv` | Consolidated AP transactions for Q1 2025 | CSV | Static snapshot (quarterly) |

**Data Source Location:** `/data/Accounts_Payable_Sample.csv`  

**Extraction Rules Summary:**
- Data is derived from AP payment systems and standardized to include payment date, due date, amount, payment status, and reprocessing info.
- Each row represents a unique **payment transaction**.
- The snapshot captures all Q1 2025 transactions (Jan–Mar).

---

## 🧩 3. Data Model

**Model Type:** Single-table structure (flat dataset)


There are **no relationships** — all DAX calculations reference this single table.

---

## 🧮 4. DAX Measures

The following measures were created in the `Measures Table` within Power BI.  
Each measure contributes directly or indirectly to visuals in the dashboard.

| **Measure Name** | **DAX Formula** | **Used in Visual / KPI?** | **Purpose** |
|------------------|------------------|----------------------------|--------------|
| **Total Volume** | `COUNTROWS('Accounts_Payable_Sample')` | ✅ | Displays total number of payment transactions; used in KPI cards and trend charts. |
| **Avg Days To Pay** | `AVERAGE('Accounts_Payable_Sample'[Days to Pay])` | ✅ | Calculates average processing time; shown as a key KPI. |
| **Avg Days To Reprocess** | `AVERAGE('Accounts_Payable_Sample'[Days to Reprocess])` | ✅ | Used to visualize and compare efficiency of reprocessed payments. |
| **On Time Payments Ratio** | `DIVIDE([On Time Payments], [Total Payments])` | ✅ | Main KPI showing share of payments completed on or before due date. |
| **On Time Target** | `0.90` | ✅ | Static target for gauge and conditional formatting visuals. |
| **Rejected Payments Ratio** | `DIVIDE([Rejected Payments], [Total Payments])` | ✅ | Tracks the proportion of rejected payments; displayed in summary cards. |
| **Rejected Target** | `0.10` | ✅ | Used as a threshold value for rejection KPIs. |
| **Rejection Reason Ratio** | `DIVIDE(COUNTROWS(FILTER('Accounts_Payable_Sample', 'Accounts_Payable_Sample'[Payment Status] = "Rejected by Bank")), [Total Payments])` | ✅ | Supports visuals explaining rejection causes and frequency. |
| **Reprocessed On Time Ratio** | `DIVIDE([Reprocessed On Time], [Reprocessed Total])` | ✅ | Displays percentage of reprocessed payments completed within the target timeframe. |

### 🧩 4.1 Calculated Columns

The following calculated columns were created in the `Accounts_Payable_Sample` table to support DAX measures and enhance filtering, time-based analysis, and KPI logic.

| **Column Name** | **DAX Formula** | **Purpose / Description** |
|------------------|------------------|-----------------------------|
| **Days To Pay** | `VAR FI_Date = 'Accounts_Payable_Sample'[Invoice Date]  
VAR Pay_Date = 'Accounts_Payable_Sample'[Payment Date]  
RETURN DATEDIFF(FI_Date, Pay_Date, DAY)` | Calculates the number of days between invoice and payment date. |
| **Days To Reprocess** | `VAR ReversalDate = 'Accounts_Payable_Sample'[Rejection Date]  
VAR SuccessDate = 'Accounts_Payable_Sample'[Reprocessed Date]  
RETURN DATEDIFF(ReversalDate, SuccessDate, DAY)` | Measures the duration between a rejected payment and its successful reprocessing. |
| **Month** | `FORMAT('Accounts_Payable_Sample'[Payment Date], "MMMM")` | Extracts the month name from the payment date for reporting and slicers. |
| **Month Number** | `MONTH('Accounts_Payable_Sample'[Payment Date])` | Numeric representation of the payment month for chronological sorting. |
| **Months** | `FORMAT('Accounts_Payable_Sample'[Payment Date], "MMM YYYY")` | Combined month-year label for slicers and timeline visuals. |
| **On Time Payment** | `IF('Accounts_Payable_Sample'[Days To Pay] <= 'Accounts_Payable_Sample'[Due Days], 1, 0)` | Flags payments completed on or before the due date. |
| **Payment Method Grouped** | `IF(  
    'Accounts_Payable_Sample'[Payment Method] IN {"Bank Transfer", "Wire"}, "Electronic",  
    IF('Accounts_Payable_Sample'[Payment Method] IN {"Cheque", "Cash"}, "Manual",  
    "Other")  
)` | Groups different payment methods into simplified reporting categories. |
| **Reprocessed On Time** | `IF('Accounts_Payable_Sample'[Days To Reprocess] <= 'Accounts_Payable_Sample'[Target Reprocess Days], 1, 0)` | Flags reprocessed payments that were completed within the target timeframe. |


---

## 🧭 5. Measure-to-Visual Mapping

| **Visual / KPI Title** | **Measure(s) Used** | **Page** | **Notes** |
|-------------------------|--------------------|-----------|------------|
| Total Payments | Total Volume | Page 1 | KPI card showing total number of AP transactions. |
| Avg Days to Pay | Avg Days To Pay | Page 1 | KPI card showing average payment cycle time. |
| On-Time Payment % | On Time Payments Ratio, On Time Target | Page 1 | KPI gauge comparing actual vs. target. |
| Rejected Payment % | Rejected Payments Ratio, Rejected Target | Page 1, 2 | KPI card with comparison indicator. |
| Monthly Payments Trend | Total Volume | Page 2 | Line chart showing payment trend by month. |
| Monthly Rejections | Rejected Payments Ratio | Page 1, 2 | Column chart visualizing rejection patterns. |
| Reprocessed On Time % | Reprocessed On Time Ratio | Page 1, 2 | KPI visual for reprocessing efficiency. |
| Slicer: Month | - | Page 2 | User-controlled slicer filtering visuals by month. |

---

## ⚙️ 6. Refresh Logic & Performance

- **Data Refresh Type:** Manual (snapshot-based, not scheduled)  
- **Source File:** CSV file uploaded to Power BI Desktop  
- **Performance Optimizations:**
  - The model includes several **calculated columns** for time-based metrics and flags (e.g., On-Time Payment, Days to Pay).  
  - Simple single-table model for demo purposes
  - Calculations are optimized to reference native date fields to maintain efficient refresh performance.
  - Static target values and grouped payment methods are handled via DAX and conditional logic for clarity and performance balance.  


---

## 🧠 7. Design & UX Notes

- Two-page layout provides a logical split:
  - **Page 1:** Executive summary of key KPIs and quarter-level results.  
  - **Page 2:** Operational monthly trends with interactive filtering.  
- Consistent color palette and layout improve readability.  
- KPI cards placed prominently to support management-level quick insights.  
- Page 2 includes slicers for user-driven exploration of monthly patterns.

---


📅 *Created October 2025*  
👤 **Author:** [Abdalla Osman]  
📍 *Data Source:* Sample Accounts Payable CSV file generated by ChatGpt, using existing ERP (SAP) standard layout   

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/abdallaosman-/)


