# Global Finance Operations Analytics

##  Project Overview
The objective of this project is to build a traceable analytics pipeline that converts operational finance data into validated KPIs, investigation flags, and dashboard-ready reporting views. 

The project evaluates three core financial processes, covering different reporting periods:
* **OTC (Order-to-Cash):** Evaluates billing, receivables, and customer payment behaviour.
* **PTP (Procure-to-Pay):** Analyses valid procurement spend and concentration.
* **RTR (Record-to-Report):** Explains revenue, expenses, profit, and account movements.

##  Architecture & Tech Stack
The delivery follows a robust five-stage workflow:
1. **Source Files:** Ingesting OTC CSVs, PTP workbooks, and RTR workbooks.
2. **Python (Jupyter):** Performs EDA, data cleaning, field standardization, and predictive forecasting.
3. **Processed CSVs:** Exports dimensions, facts, and monthly marts.
4. **PostgreSQL:** Enforces the core analytical model, referential constraints, and serves unified reporting views.
5. **Tableau:** Visualizes trends, concentration, risk, and executive performance across 16 worksheets and 4 dashboards.

##  Data Scope
* **OTC (Accounts Receivable):** 45,839 rows, 69 customers, covering £816.26M in total invoice amount (Nov 2011 to Mar 2016).
* **PTP (Procurement):** 75,349 rows yielding 61,924 valid positive-spend lines, generating £24.00B equivalent valid gross spend (Jul 2012 to Apr 2019).
* **RTR (General Ledger):** 27,909 journal lines across 3,992 unique entries and 38 mapped accounts (Jan 2018 to Dec 2020).

##  Methodology & Phases

### 1. Data Preparation & EDA (Python/Pandas)
* **Standardisation:** Cleaned column names, parsed string/Excel serial dates, and converted financial measures to appropriate numeric types.
* **Derivation:** Engineered payment delays, late-payment flags, monthly periods, and investigation flags.
* **Validation:** Reconciled processed totals back to the source data (e.g., OTC £816.26M, PTP £24.00B) to ensure zero difference. Retained zero/invalid-spend rows (13,425 records) for auditability while excluding them from spend totals.
* **Forecasting:** Built a chronological forecasting model for RTR revenue. A Linear Regression model achieved an RMSE of 153.8K, improving 21.17% upon a naive baseline (outperforming a Random Forest model on the 30-month dataset).

### 2. Relational Data Warehouse (PostgreSQL)
Separated the modelling logic from the reporting layer to prevent Tableau from repeating complex calculations.
* **Core Schema:** 5 dimensions (date, customer, material, account, territory) and 3 transaction facts.
* **Integrity:** Enforced via Primary and Foreign Keys, returning zero broken relationships across mappings.
* **Mart Layer:** Formulated 3 monthly marts and 7 analytical views (e.g., customer performance ranked by value, spend share percentages).
* **Executive Snapshot:** Unified 9 critical KPIs across OTC, PTP, and RTR into a compact management layer retaining process-specific timing context.

### 3. SQL-to-Device Handoff
Exported governed SQL views back to local CSV files to allow offline Tableau connection without exposing database credentials.

```python
from pathlib import Path
import pandas as pd

TABLEAU_DIR = Path("tableau_data")
TABLEAU_DIR.mkdir(exist_ok=True)

views = [
    "vw_executive_kpi_snapshot",
    "vw_otc_monthly_kpis",
    "vw_otc_customer_performance",
    "vw_ptp_monthly_kpis",
    "vw_ptp_material_group",
    "vw_rtr_monthly_kpis",
    "vw_rtr_account_analysis"
]

for view in views:
    df = pd.read_sql(f"SELECT * FROM mart.{view};", engine)
    df.to_csv(TABLEAU_DIR / f"{view}.csv", index=False)
```

### 4. Data Visualisation (Tableau)
Built interactive decision-ready dashboards designed for finance operations management:
* **D01 Executive KPI Snapshot:** Provides an immediate view of billing, collection efficiency, procurement scale, and financial performance margins.
* **D02 Order-to-Cash Performance & Receivables:** Highlights the divergence between billed amounts and closing AR, tracks Days Sales Outstanding (DSO), and maps customer payment risk (identifying high value vs. late payment rate combinations).
* **D03 Procure-to-Pay Spend Analytics:** Monitors valid procurement spend, highlights month-over-month growth volatility, and ranks dominant material groups (e.g., Group 1500 dominating at 15.78B) while factoring in investigation flags.
* **D04 Record-to-Report Financial Performance:** Connects revenue and expenses to net profit, detailing diverging account movements by financial-statement class and plotting journal-line volume against absolute posting sizes.

##  Conclusion
The project successfully delivers an end-to-end portfolio pipeline spanning Python, PostgreSQL, and Tableau. It ensures analytical integrity, provides a governed reporting layer, and translates raw operations into prioritized investigation workflows for financial controllers.

---
##  Author
**Tanishk Nanasaheb Shinde**
*MSc Data Science, University of Bristol*
