# Optimizing Retail Decisions through Customer Segmentation and Predictive Sales Modeling

## Executive summary
Retailers often rely on intuition for pricing and inventory. We analyze multi-table sales data and apply **customer segmentation (K-Means on RFM)** and **sales forecasting (Random Forest)** to drive decisions.

**Key outcomes**
- **Segmentation:** Two customer segments; the top segment contributes ~**74% of revenue** while representing ~**49% of customers**, enabling differentiated pricing/CRM.
- **Forecasting:** RF improves RMSE vs seasonal naive by **~15% (CV)** and **~34% (14-day holdout)**, with calibrated **~80% prediction intervals** achieving **~85.7% coverage**.
- **BI-ready outputs:** Cleaned datasets, feature artifacts, and Tableau feeds for executives.

## Methods

### Data sources
- **Primary**: Grocery Sales Dataset (Kaggle) — multi-table retail CSVs (sales, products, customers, employees, cities, countries).
- **Secondary**: Joined dimensions (e.g., products ↔ categories) enriching the sales fact table.

### Cleaning & integrity (NB01)
Deterministic pipeline standardizing types, validating foreign keys, recomputing line totals, and emitting audit artifacts.

**Row counts (after cleaning):**
| table          |    rows |
|:---------------|--------:|
| sales          | 6,758,125 |
| products       |     452 |
| cities         |      96 |
| countries      |     206 |
| customers      |   98,759 |
| employees      |      23 |
| sales_enriched | 6,758,125 |

**Overall missingness by table:**
| table          | missing_pct   |
|:---------------|:--------------|
| sales          | 0.09%         |
| products       | 0.00%         |
| cities         | 0.00%         |
| countries      | 0.00%         |
| customers      | 0.00%         |
| employees      | 0.00%         |
| sales_enriched | 0.05%         |

**Sales date range:** 2018-01-01 00:00:04.070000 → 2018-05-09 23:59:59.400000  
**FK gaps detected (counts):** sales->customers: 0, sales->products: 0, sales->employees: 0, customers->cities: 0, employees->cities: 0, cities->countries: 0

**Business logic fix:** `TotalPrice = Quantity × Price × (1 − Discount)`, materialized as `clean/sales_enriched.parquet`.

_Generated from NB01 summary on 2025-09-01 00:23:33Z UTC._

NB03 generated ML-ready features (RFM3 and an extended set), validated data contracts (schema/types), and exported artifacts used by clustering and forecasting.
#### Units & notation
- Currency: **USD**.
- In segmentation tables: `revenue` in **USD (×10k)**; `revenue_share` in **%**.
- Forecasting metrics: **RMSE/MAE in USD**, **MAPE and WAPE in %**, **R²** is unitless.


<!-- END:METHODS -->

## Introduction
**Business problem.** Grocery retail operates on thin margins; misjudged demand drives stockouts or waste—especially in perishables. Leadership needs reliable customer insights and short-horizon sales forecasts to guide promotions, staffing, and replenishment.

**Objective.** Build an end-to-end, reproducible pipeline that (1) segments customers for targeted actions and (2) forecasts daily sales with quantified uncertainty, meeting program constraints (≥2 sources; ≥1,000 rows; ≥5 columns).

**Approach.** We clean and reconcile multi-table data; engineer RFM and transactional features; run K-Means for segmentation (RFM3 and a fuller feature set); and train a Random Forest forecaster with strict walk-forward validation, a 14-day holdout, and calibrated, weekend-aware conformal prediction intervals.

**Impact.** The two-segment view concentrates value on a high-spend cohort, and the forecaster materially beats seasonal naive baselines, offering actionable PIs for operations and inventory.

## Results
### Customer Segmentation (NB04)
#### FULL — cluster mix
| cluster       |   customers | customer_share   |   revenue | revenue_share   |
|:--------------|------------:|:-----------------|----------:|:----------------|
| High-Value | 48,613 | 49.2% | 24,600 | 74.8% |
| Mid/Low-Value | 50,146 | 50.8% | 8,280 | 25.2% |
_Top cluster by revenue_: **High-Value** (revenue share ~ 74.8%, customer share ~ 49.2%).
_Notes: `revenue` in **USD (×10k)**; `revenue_share` in **%**._
#### RFM3 — cluster mix
| cluster       |   customers | customer_share   |   revenue | revenue_share   |
|:--------------|------------:|:-----------------|----------:|:----------------|
| High-Value | 46,646 | 47.2% | 23,900 | 72.8% |
| Mid/Low-Value | 52,113 | 52.8% | 8,930 | 27.2% |
_Top cluster by revenue_: **High-Value** (revenue share ~ 72.8%, customer share ~ 47.2%).
_Notes: `revenue` in **USD (×10k)**; `revenue_share` in **%**._




### Sales Forecasting (NB05) — Cross-Validation & Holdout

#### Cross-Validation (7-day WFV)
|   CV seasonal naive best RMSE |   CV RF RMSE |   CV RF MAE |   CV RF R² | CV uplift vs seasonal naive (RMSE)   |
|----------------------:|-------------:|------------:|-----------:|:-----------------------------|
|                221,907 |       187,974 |      158,439 |     -0.342 | 15.29%                       |

#### Holdout (14 days)
|   Holdout RMSE |    MAE |   MAPE (%) |   WAPE (%) |     R² | Uplift vs best seasonal naive (RMSE)   |
|---------------:|-------:|-----------:|-----------:|-------:|:-------------------------------|
|        199,746 | 161,237 |      48.00 |       0.50 | -0.114 | 33.55% |

_Note:_ MAPE can inflate on low-revenue days; for operations we prioritize **WAPE** and **RMSE**.

### Findings

Two robust customer segments with clear revenue skew (≈74% from ~49% of customers). The Random Forest forecaster improves RMSE by ~15% in cross-validation and ~34% on a 14-day holdout; PI coverage meets the ~80% target after calibration.

### Recommendations
- **CRM & pricing:** Prioritize high-value segment with tailored offers; test uplift-based discounting; monitor churn-risk within the low-value segment.
- **Inventory & staffing:** Use the 14-day forecast and PIs to set purchase orders and labor plans—especially for perishables and weekends.
- **BI adoption:** Publish the Tableau feeds (timeline & KPIs) to an executive dashboard; add alerts for forecast deviations.

### Next steps
- Expand feature space (promo/holiday/calendar effects) and evaluate additional models (LightGBM/Prophet) with the same WFV protocol.
- Move the champion model to a scheduled job; log residuals for ongoing PI recalibration.
- Enrich segmentation with CLV and promotion responsiveness for campaign optimization.

_Generated on 2025-09-01 01:14:08Z UTC_

<!-- CLOSING_REQUIREMENTS -->
**Program compliance.** The capstone satisfies program requirements (≥2 data sources; ≥1,000 rows; ≥5 columns) and demonstrates how analytical methods translate into business impact.

<!-- Patched on 2025-09-01 02:54:00Z UTC -->
