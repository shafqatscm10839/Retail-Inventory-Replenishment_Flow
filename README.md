# Retail-Inventory-Replenishment_Flow
End-to-end Python ETL and automated Excel reporting pipeline for retail inventory optimization, Ordering & Receiving of Products, DAS Calculation, OOS tracking, ABC analysis, Supplier service level, OTIF and replenishment.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data_ETL-150458?style=flat&logo=pandas&logoColor=white)
![OpenPyXL](https://img.shields.io/badge/OpenPyXL-Excel_Automation-217346?style=flat&logo=microsoft-excel&logoColor=white)
![BI Ready](https://img.shields.io/badge/Power_BI_%2F_Fabric-Ready-F2C811?style=flat&logo=powerbi&logoColor=black)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

[![Pipeline CI](https://github.com/shafqatscm10839/Retail-Inventory-Replenishment_Flow/actions/workflows/run_pipeline.yml/badge.svg)](https://github.com/shafqatscm10839/Retail-Inventory-Replenishment_Flow/actions/workflows/run_pipeline.yml)

An end-to-end **Python ETL, inventory optimization, and automated executive reporting pipeline** designed for multi-format retail operations (**Hyper**, **Super**, and **Express** stores). It replaces manual weekly spreadsheet reconciliation with a reproducible script that audits stock anomalies, calculates dynamic replenishment quantities, tracks supplier performance, and outputs BI-ready datasets.

---

## Business Problem

Retail demand planners and inventory managers routinely face four costly operational bottlenecks:

1. **Manual Reporting Overhead:** Stitching POS sales, stock-on-hand (SOH) snapshots, and supplier PO logs in Excel takes 4–6 hours per week and breaks easily when SKU counts scale.
2. **Silent Stockouts & Negative Stock:** Zero-stock items cause immediate lost revenue, while negative stock balances mask receiving errors, shrinkage, or POS scanning discrepancies.
3. **Static Reorder Rules:** Applying flat reorder thresholds across fast-moving and slow-moving SKUs causes stockouts on high-revenue items and excess cash tied up in slow movers.
4. **Unmeasured Vendor Delays:** Without automated **Supplier Service Level (SSL%)** and **OTIF** tracking, chronic short-shipments and late deliveries go unchallenged during vendor reviews.

## Solution & Operational Impact

* **30-Second Automated Reporting:** Converts raw ERP/POS exports into a styled, date-stamped, 5-tab executive Excel workbook.
* **Service-Level Driven Replenishment:** Dynamically assigns Safety Stock and Reorder Points using **ABC revenue segmentation** and **90-day demand volatility**.
* **Revenue-at-Risk Prioritization:** Ranks Out-of-Stock (OOS) and Negative Stock incidents by **Estimated Daily Lost Sales** so store teams fix the highest-impact gaps first.
* **Downstream BI Compatibility:** Exports clean, structured CSV tables ready for direct ingestion into **Microsoft Fabric Lakehouses** or **Power BI** semantic models.

---

## Pipeline Architecture

```text
+----------------------------------+
|   Raw Retail Data (data/)        |
|  - daily_sales.csv (90 Days)     |
|  - inventory_snapshot.csv        |
|  - supplier_deliveries.csv       |
+----------------+-----------------+
                 |
                 v
+----------------------------------+
|   Python ETL & Analytics Engine  |
|   (src/pipeline.py)              |
|  - Date parsing & validation     |
|  - 90-day DAS & demand std dev   |
|  - ABC revenue segmentation      |
|  - Safety Stock, ROP & Order Qty |
|  - Supplier SSL% & OTIF scoring  |
+--------+---------------+---------+
         |               |
         v               v
+------------------+   +---------------------------+
| Executive Excel  |   | BI / Fabric Ready Layer   |
| (outputs/*.xlsx) |   | (outputs/bi_*.csv)        |
| - 5 Styled Tabs  |   | - bi_inventory_master.csv |
| - RAG Highlights |   | - bi_supplier_ssl.csv     |
+------------------+   +---------------------------+



```

---

## Supply Chain Methodology & Formulas

All inventory metrics are calculated at the **Store × SKU** grain (with ABC classification evaluated across total network revenue):

| Metric | Formula / Business Logic | Purpose |
| :--- | :--- | :--- |
| **Daily Average Sales (DAS)** | `Total Units Sold (90d) / 90` | Captures true velocity across all calendar days, including Normal, Promotional & zero-sale days |
| **Demand Volatility (σ)** | Exact 90-day daily population standard deviation | Prevents underestimating variance on intermittent-demand SKUs |
| **ABC Classification** | Cumulative 90-day revenue: **A** (<= 80%), **B** (80%–95%), **C** (> 95%) | Focuses working capital and service levels on top revenue drivers |
| **Safety Stock (SS)** | `ceil(Z × σ × sqrt(Lead Time))` where Z = **1.65** (A: 95%), **1.28** (B: 90%), **0.84** (C: 80%) | Buffers against demand spikes during supplier lead time |
| **Reorder Point (ROP)** | `ceil((DAS × Lead Time) + Safety Stock)` | Triggers replenishment before stock hits zero |
| **Target Stock** | `ceil((DAS × (Lead Time + Review Cycle)) + Safety Stock)` *(Review Cycle = 7 days)* | Covers demand across lead time plus the weekly ordering cycle |
| **Suggested Order Qty** | If `SOH <= ROP`: `max(0, Target Stock - max(SOH, 0))`, else `0` | Generates actionable purchase quantities |
| **Days of Supply (DOS)** | `max(SOH, 0) / DAS` *(capped at 999 if DAS = 0)* | Flags both imminent stockouts and excess inventory (> 30 days) |
| **Est. Daily Lost Sales** | If `SOH <= 0`: `DAS × Selling Price`, else `0` | Quantifies daily top-line revenue lost per stockout |
| **Supplier Service Level (SSL%)** | `Total Delivered Qty / Total Ordered Qty` | Measures vendor volume fill rate |
| **Supplier OTIF %** | `POs Delivered On-Time AND In-Full / Total POs` | Strict vendor reliability KPI |

### Automated Stock Status Classification

* **Negative Stock** (Red): `SOH < 0` — Immediate cycle count and system reconciliation required.
* **Out of Stock** (Orange): `SOH == 0` — Active lost sales; prioritize urgent replenishment.
* **Reorder Now** (Amber): `0 < SOH <= ROP` — Below reorder threshold; include in next PO cycle.
* **Overstock** (Blue): `DOS > 30 days` — Candidate for markdown, store transfer, or order pause.
* **Healthy** (Green): Stock within optimal operating band.

---

## Automated Excel Deliverable

Running the pipeline generates `outputs/Retail_Inventory_Report_YYYY-MM-DD.xlsx` with frozen panes, auto-filters, currency/percentage formatting, and color-coded alerts across five worksheets:

1. **`Executive Summary`** — Network-wide KPI scorecard (Total Inventory Value, Combined OOS/Negative Rate, Daily Lost Sales, Replenishment Budget, Network SSL% & OTIF%) plus a breakdown by Store Format (`Hyper`, `Super`, `Express`).
2. **`OOS & Negative Stock`** — Actionable audit list sorted by highest **Estimated Daily Lost Sales**.
3. **`Replenishment Plan`** — Filtered order sheet containing only SKUs requiring replenishment, prioritized by **ABC Class** and **Replenishment Cost**.
4. **`Supplier SSL`** — Vendor scorecard with red/amber/green conditional highlighting for fill rates and OTIF below 85% and 95%.
5. **`Inventory Master`** — Complete Store × SKU analytical table for ad-hoc filtering and deep dives.

---

## Repository Structure

```text
Retail-Inventory-Replenishment_Flow/
├── data/                   # Synthetic raw input CSVs (sales, inventory, deliveries)
├── src/
│   ├── generate_data.py    # Reproducible synthetic retail dataset generator
│   └── pipeline.py         # Core ETL, analytics engine, and OpenPyXL report builder
├── outputs/                # Generated Excel reports & BI-ready CSV exports
├── requirements.txt        # Python dependencies
├── LICENSE                 # MIT License
└── README.md               # Project case study & documentation
```

---

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/shafqatscm10839/Retail-Inventory-Replenishment_Flow.git
cd Retail-Inventory-Replenishment_Flow
```

### 2. Set Up Environment & Install Dependencies
```bash
conda create -n retail_scm python=3.11 -y
conda activate retail_scm
pip install -r requirements.txt
```

### 3. Generate Synthetic Data & Run the Pipeline
```bash
python src/generate_data.py
python src/pipeline.py
```

The date-stamped Excel report and BI-ready CSV files will appear inside the `outputs/` folder.

---

## Data Privacy & Disclaimer

All datasets generated and processed in this repository are **100% synthetic** and created programmatically via `src/generate_data.py` to model realistic retail supply chain behavior without exposing proprietary or confidential business data.

---

## About the Author

**Shafqat Hussain**  
Supply Chain & Demand Planning Manager | Microsoft Fabric & BI Data Analyst  

Specializing in retail demand planning, inventory optimization, Python automation, SQL, Power BI, and Microsoft Fabric analytics pipelines.

* **LinkedIn:** [linkedin.com/in/shafqat-hussain-scm](https://linkedin.com/in/shafqat-hussain-scm)
* **GitHub:** [github.com/shafqatscm10839](https://github.com/shafqatscm10839)
