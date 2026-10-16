# Olist Delivery Performance Analytics

**Delivery KPI Analysis · Root Cause Analysis · Late-Delivery Risk Prediction**

An end-to-end analytics project using the [Brazilian E-Commerce Public Dataset by Olist on Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce). The project investigates delivery performance, identifies potential operational bottlenecks, examines the association between delivery delays and customer reviews, and evaluates whether late-delivery risk can be flagged at checkout.

> **Dataset scope:** The dataset represents a Brazilian marketplace from 2016–2018. Findings describe the analyzed dataset and should not be interpreted as current delivery performance in Indonesia or another marketplace.

## Project Overview

Delivery delays can occur at different stages of the order journey. This project examines seller handover, carrier transit, payment/platform approval, and delivery estimates to understand where delays may arise.

The analysis combines SQL and Python, statistical analysis, and a time-based machine-learning evaluation. It also prepares analysis-ready tables for a BI dashboard.

## Business Questions

1. What are the on-time delivery rate, late delivery rate, and typical delivery duration?
2. Which stage of the delivery process contributes most to delays?
3. How do delivery outcomes vary by customer region, seller, route, and product category?
4. How are delivery delays associated with customer review scores after accounting for selected factors?
5. Can late-delivery risk be identified at checkout using information available at that point?

## Key Findings

The analysis base contains **95,082 delivered orders** after excluding records with missing timestamps or invalid stage chronology.

| Metric | Result |
|---|---:|
| Orders analyzed | 95,082 |
| On-time delivery rate | 93.15% |
| Late delivery rate | 6.85% |
| Average delivery time | 12.62 days |
| Median delivery time | 10.27 days |
| Average delay among late orders | 10.62 days |

### Operational Insights

- Carrier transit was the primary attributed driver for **84.04% of late orders** in the distance-adjusted root-cause analysis.
- Seller handover accounted for **15.35% of late orders** in the same analysis.
- The Northeast region had a **12.81% late rate** among the regional groups included in the analysis.
- A seller Pareto analysis found that **67 sellers (2.3% of sellers in the analyzed scorecard) accounted for 50% of seller-driven late orders**.
- In a controlled regression, late delivery was associated with a **2.01-star lower review score** (95% CI: −2.05 to −1.97). This is an observational association, not proof of causation. A sensitivity analysis limited to reviews created after delivery estimated a smaller association of −0.77 stars (95% CI: −0.84 to −0.70).

### Late-Delivery Risk Model

A LightGBM model was evaluated using a **time-based split**, with test data from orders purchased on or after `2018-04-01`. The model uses checkout-available features to avoid relying on delivery-stage information that would not be known at prediction time.

| Evaluation Metric | Result |
|---|---:|
| ROC-AUC | 0.722 |
| PR-AUC | 0.120 |
| Test-set late-order rate (PR-AUC baseline) | 0.042 |

Among the **top 20% highest-risk orders**, the model captured **48.02% of late orders** in the test set, with **10.20% precision**. This suggests a possible prioritization use case; it is not a production-ready deployment.

## Methodology

1. **Data audit** — inspected row counts, order statuses, missing timestamps, duplicate order IDs, and one-to-many relationships.
2. **Data cleaning** — focused on delivered orders with complete timestamps and valid chronological delivery stages; documented exclusions in a cleaning funnel.
3. **SQL analysis** — used SQLite joins, common table expressions (CTEs), aggregations, and window functions to calculate KPIs and seller-level metrics.
4. **Feature engineering** — calculated approval, seller-handover, transit, total-delivery, delay, distance, and estimate-buffer features.
5. **Root cause analysis** — compared delayed orders with on-time benchmarks, including distance-binned transit benchmarks. These comparisons help make transit attribution more comparable across routes.
6. **Statistical analysis** — examined review-score differences using controlled regression and a sensitivity analysis for reviews created after delivery.
7. **Machine learning** — trained and evaluated LightGBM with a time-based split and checkout-time features; assessed ranking utility at different risk thresholds.
8. **Dashboard data export** — generated CSV tables for BI reporting.

## Tech Stack

- **Language & data analysis:** Python, pandas, NumPy
- **SQL & database:** SQL, SQLite
- **Statistics & visualization:** statsmodels, Matplotlib, Seaborn
- **Machine learning:** LightGBM, scikit-learn metrics, SHAP-style feature attribution using LightGBM TreeSHAP contributions
- **BI preparation:** CSV exports designed for Looker Studio or Power BI

## Repository Structure

```text
olist-delivery-analytics/
├── README.md
├── notebooks/
│   └── oilisttt-delivery-analytics.ipynb
├── sql/
│   └── SQL queries exported by the notebook
├── dashboard_data/
│   └── KPI and analysis-ready CSV exports
├── figures/
│   └── Generated charts
├── src/
│   └── ETL modules (planned)
├── requirements.txt
└── .gitignore
```

This is a **suggested structure**. Keep only files that actually exist in the repository and update this section as the project evolves. The notebook currently generates SQL queries, figures, and dashboard data in its working directory; organize these outputs into the folders above when setting up the repository.

## Getting Started

### 1. Download the Dataset

Download the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) from Kaggle and extract the CSV files into a local data directory.

The notebook uses files from the Olist dataset, including:

- `olist_orders_dataset.csv`
- `olist_customers_dataset.csv`
- `olist_order_items_dataset.csv`
- `olist_order_reviews_dataset.csv`
- `olist_products_dataset.csv`
- `olist_sellers_dataset.csv`
- `olist_geolocation_dataset.csv`
- `product_category_name_translation.csv`

The notebook includes Kaggle-specific input-path discovery. For local execution, update the data-path configuration to point to the directory containing the extracted files.

### 2. Install Dependencies

Create and activate a Python environment, then install the packages used by the notebook:

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn lightgbm shap jupyter
```

### 3. Run the Notebook

Open the notebook in Jupyter Notebook or VS Code and run the cells in order. The notebook creates output folders for figures, SQL queries, and dashboard data. Exact output paths depend on the environment configuration.

## Dashboard

**Status:** In progress.

## Limitations

- **Historical and geographic scope:** the dataset covers a Brazilian marketplace from 2016–2018. Results may not represent current conditions or another market.
- **Root-cause attribution is heuristic:** primary drivers are assigned using benchmark comparisons, including distance-binned transit benchmarks. These are analytical attributions, not confirmed operational incident records.
- **Review analysis is observational:** regression controls account for selected factors but do not establish that delays alone caused lower ratings.
- **Model generalization:** performance is measured on a historical time-based test split from the same dataset. Further validation is required before real-world use.
- **Class imbalance:** late deliveries are a minority class, so PR-AUC and top-risk capture are reported alongside ROC-AUC.
- **Not production-ready:** the model is an analytical prototype, not a deployed decision system.


Download the original dataset from the source page and follow the dataset provider's terms when using or redistributing the data. This repository is intended for analysis code and appropriate derived outputs, not redistribution of the raw dataset.

---

*Project focus: data analytics, Python*
