# Retail-inventory-replenishment-engine
End-to-end Python ETL and automated Excel reporting pipeline for retail inventory optimization, Ordering & Receiving of Products, OOS tracking, ABC analysis, Supplier service level, OTIF and replenishment.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data_ETL-150458?style=flat&logo=pandas&logoColor=white)
![OpenPyXL](https://img.shields.io/badge/OpenPyXL-Excel_Automation-217346?style=flat&logo=microsoft-excel&logoColor=white)
![BI Ready](https://img.shields.io/badge/Power_BI_%2F_Fabric-Ready-F2C811?style=flat&logo=powerbi&logoColor=black)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

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
