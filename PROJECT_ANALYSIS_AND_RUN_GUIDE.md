# FMCG Market Analytics Project - Complete Analysis and Run Guide

Date: 16 April 2026

## 1) What this project is (in simple terms)

This is a Databricks data engineering project for an FMCG business use case.

The objective is to combine sales data from:
- A parent company dataset
- A child company dataset (including daily incremental files)

and produce analytics-ready tables for BI dashboards (Power BI).

It uses a Medallion/Lakehouse pattern:
- Bronze: raw ingested data
- Silver: cleaned and standardized data
- Gold: business-ready fact and dimension tables + enriched reporting view

## 2) What problem it solves

When companies merge/acquire, data is fragmented across systems and formats.
This project standardizes and unifies those sources so business teams can track:
- Sales trends
- Product performance
- Region/market/channel performance
- Total revenue metrics

## 3) Core architecture and flow

Main logical flow:
1. Load raw files from 0_data sources into Bronze Delta tables.
2. Clean and transform dimensions (customers, products, gross price) into Silver/Gold dimension tables.
3. Process fact orders (full load + incremental load) into Silver and Gold fact tables.
4. Create denormalized Gold view for BI.
5. Connect Power BI to Databricks and build dashboard.

Reference query for reporting view:
- 2_dashboarding/denormalise_table_query_fmcg.txt
- Creates: fmcg.gold.vw_fact_orders_enriched

## 4) Important folders in this repository

- 0_data/
  - Parent and child source files (CSV) for full and incremental loads.
- 1_codes/
  - Databricks notebooks for setup, dimensions, and fact pipelines.
- 2_dashboarding/
  - Gold-layer denormalization SQL and dashboard artifacts.
- Scripts/
  - Local Python helper scripts for data conversion and scaling.

## 5) Notebook execution order (how to run correctly)

Run in Databricks in this sequence:

1. 1_codes/1_setup/setup_catalog.ipynb
2. 1_codes/1_setup/dim_date_table_creation.ipynb
3. 1_codes/2_dimension_data_processing/1_customers_data_processing.ipynb
4. 1_codes/2_dimension_data_processing/2_products_data_processing.ipynb
5. 1_codes/2_dimension_data_processing/3_pricing_data_processing.ipynb
6. 1_codes/3_fact_data_processing/1_full_load_fact.ipynb
7. 1_codes/3_fact_data_processing/2_incremental_load_fact.ipynb
8. 2_dashboarding/denormalise_table_query_fmcg.txt (run as SQL in Databricks)

Then connect Power BI to query fmcg.gold.vw_fact_orders_enriched.

## 6) What each stage does

Setup:
- Creates catalog/schema context (fmcg bronze/silver/gold style organization).
- Creates date dimension in Gold (fmcg.gold.dim_date).

Dimensions:
- Customers: cleans and standardizes customer attributes.
- Products: adds business grouping fields (division/category/variant), normalizes IDs.
- Pricing: normalizes month and price values.

Facts:
- Full load notebook ingests historical orders and moves processed files from landing.
- Incremental notebook ingests newly arrived order files and merges/upserts into target tables.

Gold view:
- Joins fact_orders + dim_date + dim_customers + dim_products + dim_gross_price.
- Adds derived metric total_amount_inr.

## 7) Prerequisites to run

Recommended environment:
- Databricks workspace (on AWS, Azure, or GCP)
- Spark cluster/SQL warehouse
- Access to storage location for raw data (DBFS mount, S3, or external location)
- Permissions to create schemas/tables/views

Local tools (optional, for helper scripts only):
- Python 3.10+
- pip install pandas pyarrow tqdm

## 8) Key configuration notes before running

1. Validate all paths inside notebooks:
   - Some notebooks include hardcoded S3-like paths (example pattern: s3://...).
   - Update to your own storage path.

2. Validate catalog/schema/table names:
   - Confirm catalog defaults and schema names are consistent.

3. Confirm landing/processed folder strategy:
   - Fact notebooks move files from landing to processed.
   - Ensure folder permissions and paths are correct.

4. Validate data types before production:
   - Some logic maps invalid customer/product IDs to fallback values.
   - Verify this aligns with your data governance policy.

## 9) How to run locally vs Databricks

This is primarily a Databricks notebook project.
There is no single local web server command like npm start.

Local execution is mainly for helper scripts:

A) Convert incremental CSV to Parquet:
- Script: Scripts/convert_orders_to_parquet.py
- Note: currently contains machine-specific absolute paths.
- Update source_dir and dest_dir first.

B) Scale child company data for load testing:
- Script: Scripts/duplicate_child_data.py
- Example:
  python3 Scripts/duplicate_child_data.py --multiplier 10 --dry-run

## 10) What to improve before production

- Replace hardcoded paths with environment/config parameters.
- Add one orchestration entrypoint (Databricks Workflow job JSON or Terraform).
- Add data quality checks (null, duplicate, schema drift alerts).
- Add observability (row counts, reject counts, SLA logs).
- Add CI checks for notebook linting/versioning where possible.

## 11) How to make this project live (deployment options)

Important: This is a data pipeline project, not a standalone frontend app.
So "make live" generally means one or both of these:

Option A: Live Data Pipeline + Live Dashboard (recommended)
1. Deploy notebooks to Databricks workspace.
2. Create Databricks Workflow Job with task dependencies.
3. Schedule full load once and incremental loads daily/hourly.
4. Create/refresh Gold view.
5. Connect Power BI (or Databricks SQL dashboard) to Gold view.
6. Publish dashboard to Power BI Service and share with stakeholders.

Option B: Portfolio/Showcase live
1. Keep pipeline in GitHub.
2. Add architecture diagram + sample dashboard screenshots.
3. Add reproducible setup docs and sample outputs.
4. Optionally host a static project page (GitHub Pages/Notion) linking repo + dashboard snapshots.

## 12) Suggested Databricks Job task dependency chain

- Task 1: setup_catalog (manual/once)
- Task 2: dim_date_table_creation (manual/once)
- Task 3: customers_processing
- Task 4: products_processing
- Task 5: pricing_processing
- Task 6: full_load_fact (initial run only)
- Task 7: incremental_load_fact (scheduled)
- Task 8: denormalized_gold_view_refresh

## 13) Risks and caveats I observed

- Hardcoded storage paths in notebooks can break portability.
- Local conversion script has absolute Windows paths currently.
- There is no one-click Infra-as-Code deployment included.
- Data contracts/schema validation are implicit rather than strongly enforced in code.

## 14) Quick start checklist

- [ ] Import notebooks to Databricks
- [ ] Upload/mount 0_data paths
- [ ] Update storage paths + catalog settings
- [ ] Run setup + dimensions + facts in sequence
- [ ] Run Gold view SQL query
- [ ] Connect and publish dashboard
- [ ] Schedule incremental workflow

---

If you want, next I can generate for you:
1) A production-grade Databricks Workflow JSON spec,
2) A parameterized config template (dev/test/prod), and
3) A polished deployment checklist for interview/demo use.
