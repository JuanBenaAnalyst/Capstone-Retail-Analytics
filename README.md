# Capstone-Retail-Analytics
Capstone Project Professional Certificate in Data Analytics

**Optimizing Retail Decisions through Customer Segmentation and Predictive Sales Modeling**

**Table of Contents**

- [Executive summary](#executive-summary)
- [Introduction](#introduction)
- [Methods](#methods)
- [Results](#results)
- [Conclusion](#conclusion)
- [Appendix — Technical snapshots (NB01–NB05)](#appendix-technical)
- [Appendix — Artifact snapshot](#appendix-artifacts)

<a id="executive-summary"></a>

## Executive summary

- **Data sources (≥2)**: ✓
- **Rows (≥1,000)**: ✓
- **Columns (≥5)**: ✓

- We analyzed **6,690,599 transactions** (2018-01-01→2018-05-09) joined with product and customer–city–country dimensions.
- EDA shows **ATV ≈ 641.07** (median **490.77**, P90 **1,472.30**) and **UPT ≈ 13** (median **13**, P90 **23**).
- Mix: perishables ≈ **88.7%**; top category **Produce**. Geographic revenue is **Low** concentration (HHI 0.010, Top-5 5.6%, Gini 0.017).
- Segmentation selects **k=2** (silhouette **0.382** vs **0.253**; DBI **1.007** vs **1.357**; CH **87,886** vs **63,005**). **Cluster 0** (High-value core) drives ~**73.7%** of revenue.
- Random-Forest outperforms the **seasonal naïve** by **~15.3% CV RMSE** and **~33.6% on 14-day holdout** (holdout MAE **161,237**; R² CV **-0.342**, holdout **-0.114**).
- Conformal prediction intervals achieve **~85.71% coverage** at an 80% target (mean width **520,187.57**).
- **Recommendations:** target the high-value cluster with tailored pricing/assortment, monitor perishable categories, and use the forecast with PIs for inventory & staffing; refresh segments monthly and roll forecasts weekly with automatic data-quality checks.
*Operational note:* **WAPE** is aggregation-weighted; values can be much lower than **MAPE**. Track WAPE weekly.
<a id="introduction"></a>

## Introduction

Retail decision-makers need clear answers to **who buys** (customer segments) and **what comes next** (short-horizon sales forecasts) to guide promotions, inventory levels, and staffing. We address the question:

**“How can customer segmentation and short-horizon sales forecasting improve targeting and operational planning for this retail dataset?”**

This is well-suited to a **data-driven** approach because we have itemized transactions, product attributes, and geographies over a continuous time window. These support robust descriptive analytics, data-validated segments, and predictive models with actionable uncertainty bands.
<a id="methods"></a>

## Methods


### Data sources (at least two)

- (1) **Sales transactions** — 6,690,599 rows, 15 columns
- (2) **Customers** — 98,759 rows, 16 columns
- (3) **Daily sales aggregate** — 129 rows, 3 columns
- **Currency:** All monetary figures reported in USD.

### Data cleaning & feature engineering

- Safe numeric casting and type normalization across joins.
- Rows with missing `SalesDate` are **excluded** from time-series (~99.90%) but **kept** for global KPIs.
- RFM reference date set to dataset max date (2018-05-09).
- Descriptives trimmed at **P99** to mitigate heavy tails.
- Scaled features maintained under a **feature catalog** contract for downstream models.

### Modeling & justification

- **Segmentation:** K-Means on scaled features (RFM + add-ons like avg ticket, %discounted, #categories). k=2 maximized silhouette, minimized DBI, and improved CH.
- **Forecasting:** Random Forest with forward-computable features; **seasonal naïve** baseline. Validation via repeated CV and a 14-day holdout; uncertainty via weekend-aware conformal prediction intervals targeting 80% coverage.

### Assumptions & limitations

- **Time window** limited to observed period; patterns may differ outside it.
- **Missing promotions/calendar** variables; forecasts may miss uplift/dips from events or price changes.
- **Labeling assumptions** (perishable/class/category) may contain errors; monitor drift and re-validate monthly.
<a id="results"></a>

## Results


### Descriptive statistics (selected KPIs)

| Metric                               | Value                      |
|:-------------------------------------|:---------------------------|
| ATV (mean / median / P90)            | 641.07 / 490.77 / 1,472.30 |
| UPT (mean / median / P90)            | 13 / 13 / 23               |
| Recency (median / P75, days)         | 1 / 2                      |
| Frequency (median / P75)             | 68 / 73                    |
| Monetary (median / P75)              | 42,557 / 63,148            |
| Perishables share                    | 88.7%                      |
| HHI (class)                          | 0.010 (Low)                |
| Top-5 city share                     | 5.6%                       |
| Gini                                 | 0.017                      |
| Pareto (% customers for 80% revenue) | 55.9%                      |
| Rows excluded from time series       | 99.90%                     |


### Customer segmentation

Model selection snapshot (k-scan):

|   k |   silhouette |   davies_bouldin |   calinski_harabasz |
|----:|-------------:|-----------------:|--------------------:|
|   2 |       0.3819 |           1.0071 |             87885.7 |
|   3 |       0.2532 |           1.357  |             63004.6 |


**Cluster labels (executive):** Cluster 0 = High-value core; Cluster 1 = Value-seekers.

**Executive profile (medians, FULL)**

|   cluster_full |   recency_days_scaled_median |   frequency_scaled_median |   monetary_scaled_median |   avg_ticket_scaled_median |   pct_discounted_scaled_median |   n_categories_scaled_median | share   | count   |
|---------------:|-----------------------------:|--------------------------:|-------------------------:|---------------------------:|-------------------------------:|-----------------------------:|:--------|:--------|
|              0 |                       0.0476 |                    0.4848 |                   0.4938 |                     0.5863 |                         0.4249 |                       0.6154 | 49.2%   | 48,613  |
|              1 |                       0.0476 |                    0.4697 |                   0.1641 |                     0.1919 |                         0.4311 |                       0.6154 | 50.8%   | 50,146  |


**Revenue by cluster (FULL)**

|   cluster_full | revenue_usd   | n_customers   |   freq_mean |   recency_days_median | revenue_share   |   lift_mon_per_cust |
|---------------:|:--------------|:--------------|------------:|----------------------:|:----------------|--------------------:|
|              0 | $3.16B        | 48,613        |     68.3033 |                     1 | 73.7%           |            0.4967   |
|              1 | $1.13B        | 50,146        |     67.2072 |                     1 | 26.3%           |           -0.481516 |


### Sales forecasting performance

- **CV — seasonal naïve RMSE (mean)**: 221,907
- **CV — RF RMSE (mean)**: 187,974
- **CV — RF MAE (mean)**: 158,439
- **CV — RF R² (mean)**: -0.342
- **CV — Uplift vs seasonal naïve (RMSE)**: 15.3%
- **Holdout (14d) — RMSE**: 199,746
- **Holdout (14d) — MAE**: 161,237
- **Holdout (14d) — MAPE (%)**: 48.22%
- **Holdout (14d) — WAPE (%)**: 0.48%
- **Holdout (14d) — R²**: -0.114
- **Holdout — Uplift vs seasonal naïve (RMSE)**: 33.6%
- **Holdout predictions (rows)**: 14
- **Future forecast (rows)**: 14

*Note:* R² can be less informative on seasonal time series. The key benchmark here is the seasonal naïve; the Random Forest improves RMSE by the percentages reported above (CV and holdout).

*Note:* **MAPE** can inflate on low-sales days. For inventory and staffing we emphasize **WAPE (%)** as the primary operational metric.

**Operational KPI:** track **WAPE (%) weekly** with service-level monitoring.

### Prediction intervals

- **Prediction interval coverage (holdout, target 80%)**: 85.71%
- **Mean PI width (holdout)**: 520,187.57

Use the **upper PI** as a safety-stock buffer for perishables and weekend peaks.

### Figures

<details>
<summary>Show figures</summary>

<img src="nb02/plots/sales_monthly_line.png" width="720" alt="Total Sales by Month">
<br><sub><b>Total Sales by Month.</b> Monthly total sales (rows with valid SalesDate only; policy excludes null dates).</sub>

<img src="nb02/plots/sales_by_weekday_bar.png" width="720" alt="Total Sales by Day of Week">
<br><sub><b>Total Sales by Day of Week.</b> Total sales aggregated by weekday (valid dates only).</sub>

<img src="nb02/plots/ts_included_vs_excluded.png" width="720" alt="Rows Included vs Excluded (Time-Series Policy)">
<br><sub><b>Rows Included vs Excluded (Time-Series Policy).</b> Rows included vs excluded in time-series computations, per date-policy.</sub>

<img src="nb02/plots/sales_totalprice_hist.png" width="720" alt="TotalPrice (<= P99: 2,130.44)">
<br><sub><b>TotalPrice (<= P99: 2,130.44).</b> Distribution of TotalPrice trimmed at the 99th percentile (P99=2,130.44) to reduce extreme values.</sub>

<img src="nb02/plots/sales_totalprice_box.png" width="720" alt="TotalPrice (boxplot)">
<br><sub><b>TotalPrice (boxplot).</b> Boxplot of TotalPrice; whiskers show IQR-based spread and highlight outliers for quality checks.</sub>

<img src="nb02/plots/sales_quantity_hist.png" width="720" alt="Quantity (<= P99: 25.00)">
<br><sub><b>Quantity (<= P99: 25.00).</b> Distribution of Quantity trimmed at the 99th percentile (P99=25.00) to reduce extreme values.</sub>

<img src="nb02/plots/sales_quantity_box.png" width="720" alt="Quantity (boxplot)">
<br><sub><b>Quantity (boxplot).</b> Boxplot of Quantity; whiskers show IQR-based spread and highlight outliers for quality checks.</sub>

<img src="nb02/plots/sales_discount_hist.png" width="720" alt="Discount (<= P99: 0.20)">
<br><sub><b>Discount (<= P99: 0.20).</b> Distribution of Discount trimmed at the 99th percentile (P99=0.20) to reduce extreme values.</sub>

<img src="nb02/plots/sales_discount_count.png" width="720" alt="Discount distribution (discrete levels)">
<br><sub><b>Discount distribution (discrete levels).</b> Frequency of discrete discount levels (e.g., 0%, 10%, 20%); useful to audit pricing ladders.</sub>

<img src="nb02/plots/products_price_hist.png" width="720" alt="Price (<= P99: 98.35)">
<br><sub><b>Price (<= P99: 98.35).</b> Distribution of Price trimmed at the 99th percentile (P99=98.35) to reduce extreme values.</sub>

<img src="nb02/plots/products_price_box.png" width="720" alt="Price (boxplot)">
<br><sub><b>Price (boxplot).</b> Boxplot of Price; whiskers show IQR-based spread and highlight outliers for quality checks.</sub>

<img src="nb02/plots/products_class_counts.png" width="720" alt="Products by Class">
<br><sub><b>Products by Class.</b> Number of products by Class; helps explain assortment composition for executive stakeholders.</sub>

</details>

- `nb02/plots/products_perishable_counts.png` — **Perishable vs Non-Perishable Products**. Count of products by Perishable flag (1=perishable, 0=non-perishable); informs inventory policy.
- `nb02/plots/sales_top_categories_by_revenue.png` — **Top Categories by Revenue**. Top categories by revenue (bars sorted descending).
- `nb02/plots/sales_city_by_month_heatmap.png` — **City × Month Revenue (Top Cities)**. City × Month revenue heatmap (top revenue cities; valid dates only).
- `nb02/plots/rfm_recency_(days)_hist.png` — **Recency (days) (<= P99: 8)**. RFM Recency (days): distribution trimmed at the 99th percentile (P99=8) to reduce extreme values. Recency is computed as days since the customer's last purchase relative to the dataset reference date.
- `nb02/plots/rfm_frequency_(transactions)_hist.png` — **Frequency (transactions) (<= P99: 87)**. RFM Frequency (transactions): distribution trimmed at the 99th percentile (P99=87) to reduce extreme values. Recency is computed as days since the customer's last purchase relative to the dataset reference date.
- `nb02/plots/rfm_monetary_(total_spend)_hist.png` — **Monetary (total spend) (<= P99: 96,911)**. RFM Monetary (total spend): distribution trimmed at the 99th percentile (P99=96,911) to reduce extreme values. Recency is computed as days since the customer's last purchase relative to the dataset reference date.
- `nb02/plots/rfm_correlation_heatmap.png` — **RFM Correlation**. Correlation matrix for RFM features (Recency, Frequency, Monetary). Typical retail patterns: negative correlation between Recency and Frequency/Monetary (more recent buyers tend to buy more).
- `nb02/plots/pareto_customers_revenue.png` — **Pareto Curve — ~55.9% of customers drive 80% of revenue**. Pareto curve: share of customers vs share of revenue (red lines indicate the 80% threshold).

<a id="conclusion"></a>

## Conclusion

- **Prioritize Cluster 0** with tailored promotions, availability, and cross-sell; develop uplift tests for Cluster 1.
- **Operationalize PIs** in replenishment and staffing: use upper PI bounds for perishables and weekend peaks.
- **Model refinement**: add calendar/events/promotions and price indices; evaluate gradient boosting as a challenger.
- **Governance**: monthly segment refresh; weekly forecast roll with automated data quality checks; track WAPE/MAPE and service levels.
- **Playbooks**: build city/category tactics given low geo concentration but meaningful perishable share.
- **Pilot**: in **Produce** for **Cluster 0 (High-value core)**, tune price/assortment + on-shelf availability; monitor **ATV**, **WAPE (%)**, and **service level** over 2 weeks.
*Impact:* Using PI-informed replenishment for perishables can reduce stockouts and waste while aligning staffing to demand uncertainty bands.

**Owner:** Analytics Team · **Next refresh:** weekly (Fridays EOD) · **Data window:** 2018-01-01→2018-05-09

## How to reproduce

Minimal commands to rebuild inputs, figures, models and this report:

```bash
# 1) Build inputs & EDA
python pipelines/nb01_build_inputs.py
python pipelines/nb02_eda.py
# 2) Clustering & Forecast
python pipelines/nb04_cluster.py
python pipelines/nb05_forecast.py
# 3) Generate Executive Report (this README)
python readme_builder.py
```


<a id="appendix-technical"></a>

## Appendix — Technical snapshots (NB01–NB05)



## NB01 — Data Ingestion & Standardization

Summarizes standardized model inputs for downstream notebooks.

**Customers base**

- **rows**: 98,759
- **unique IDs**: 98,759
- **columns**: 16

| column                |
|:----------------------|
| CustomerID            |
| last_date             |
| frequency             |
| monetary              |
| recency_days          |
| avg_ticket            |
| pct_discounted        |
| n_categories          |
| top_city              |
| top_country           |
| recency_days_scaled   |
| frequency_scaled      |
| monetary_scaled       |
| avg_ticket_scaled     |
| pct_discounted_scaled |
| n_categories_scaled   |

**Sales daily series**

- **date range**: 2018-01-01 → 2018-05-09
- **rows**: 129
- **cols**: 3

**Sales transactions**

- **rows**: 6,690,599
- **cols**: 15
- **date range**: 2018-01-01 → 2018-05-09

**Feature catalog**

- **features_full**: 6
- **features_rfm3**: 3

Top features (FULL):

| feature               |
|:----------------------|
| recency_days_scaled   |
| frequency_scaled      |
| monetary_scaled       |
| avg_ticket_scaled     |
| pct_discounted_scaled |
| n_categories_scaled   |

Features (RFM3):

| feature             |
|:--------------------|
| recency_days_scaled |
| frequency_scaled    |
| monetary_scaled     |



## NB04 — Customer Clustering (GPU/CPU)

Key clustering artifacts and quick stats.

- **labeled customers**: 98,759
- **k (FULL)**: 2
- **HV share (FULL)**: 49.2%

Quick K-scan (FULL) — silhouette:

|   k |   silhouette |
|----:|-------------:|
|   2 |       0.3789 |
|   3 |       0.2504 |
|   4 |       0.2095 |
|   5 |       0.1995 |
|   6 |       0.1783 |
|   7 |       0.1633 |
|   8 |       0.1607 |
|   9 |       0.1582 |
|  10 |       0.1547 |

Confirmed candidates (FULL):

|   k |   inertia |   silhouette |   davies_bouldin |   calinski_harabasz |
|----:|----------:|-------------:|-----------------:|--------------------:|
|   2 |   7598.87 |       0.3819 |           1.0071 |             87885.7 |
|   3 |   6310.07 |       0.2532 |           1.357  |             63004.6 |

Executive profile (FULL) — medians:

|   cluster_full |   recency_days_scaled_median |   frequency_scaled_median |   monetary_scaled_median |   avg_ticket_scaled_median |   pct_discounted_scaled_median |   n_categories_scaled_median | share   | count   |
|---------------:|-----------------------------:|--------------------------:|-------------------------:|---------------------------:|-------------------------------:|-----------------------------:|:--------|:--------|
|              0 |                       0.0476 |                    0.4848 |                   0.4938 |                     0.5863 |                         0.4249 |                       0.6154 | 49.2%   | 48,613  |
|              1 |                       0.0476 |                    0.4697 |                   0.1641 |                     0.1919 |                         0.4311 |                       0.6154 | 50.8%   | 50,146  |

Revenue by cluster (FULL):

|   cluster_full | revenue_usd   | n_customers   |   freq_mean |   recency_days_median | revenue_share   |   lift_mon_per_cust |
|---------------:|:--------------|:--------------|------------:|----------------------:|:----------------|--------------------:|
|              0 | $3.16B        | 48,613        |     68.3033 |                     1 | 73.7%           |            0.4967   |
|              1 | $1.13B        | 50,146        |     67.2072 |                     1 | 26.3%           |           -0.481516 |



## NB05 — Sales Forecasting (V2)

Champion RF model with forward-computable features, cross-validated and holdout-tested.

**Cross-Validation (CV)**

- **seasonal naïve RMSE (mean)**: 221,907
- **RF RMSE (mean)**: 187,974
- **RF MAE (mean)**: 158,439
- **RF R² (mean)**: -0.342
- **Uplift vs seasonal naïve (RMSE)**: 15.3%

**Holdout (14d)**

- **RMSE**: 199,746
- **MAE**: 161,237
- **MAPE (%)**: 48.22%
- **WAPE (%)**: 0.48%
- **R²**: -0.114
- **Uplift vs seasonal naïve (RMSE)**: 33.6%

Holdout predictions: **14** rows.

Future forecast: **14** rows.

Prediction intervals (conformal, weekend-aware, 80% target):

- **coverage% (holdout)**: 85.71
- **mean width**: 520,187.57


<a id="appendix-artifacts"></a>

## Appendix — Artifact snapshot

- `clean/forecast/models/rf_champion_forward.joblib`
- `clean/forecast/predictions/champion_backtest_residuals.parquet`
- `clean/forecast/predictions/rf_holdout_predictions_forward.parquet`
- `clean/forecast/predictions/rf_future_forecast_forward.parquet`
