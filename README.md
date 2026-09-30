# Medallion Data Warehouse — SQL Server
 
Building a modern data warehouse with **SQL Server**, implementing a **Medallion Architecture** (Bronze, Silver, Gold layers) to consolidate sales data from two source systems — **CRM** and **ERP** — into a clean, analytics-ready **star schema**.
 
This project covers the full pipeline: data ingestion, cleansing, transformation, modeling, and quality validation, along with supporting documentation such as a data catalog and naming conventions guide.
 
---
 
## Data Architecture
 
The warehouse follows the Medallion Architecture with three layers:
 
<img width="1544" height="795" alt="data_architecture" src="https://github.com/zshafique25/Medallion-Data-Warehouse-SQL-Server/blob/master/docs/1_data_architecture.png" />
 
- **🥉 Bronze Layer** — Raw, unprocessed data ingested as-is from source CSV files (CRM & ERP) into SQL Server via `BULK INSERT`. No transformations are applied; this layer serves as the single source of truth for raw data.
- **🥈 Silver Layer** — Cleaned and standardized data. Includes deduplication, trimming whitespace, normalizing codes into readable values (e.g., `M` → `Married`), handling invalid dates/prices, and enriching records — preparing the data for modeling.
- **🥇 Gold Layer** — Business-ready data modeled into a **star schema** (dimension and fact views) that's directly consumable for reporting and analytics.
---
 
## Overview
 
This project demonstrates a complete, end-to-end data warehousing workflow:
 
1. **Data Architecture** — Designing a Bronze, Silver, and Gold layered warehouse.
2. **ETL Pipelines** — Extracting, cleansing, and loading data from source systems using T-SQL stored procedures.
3. **Data Modeling** — Building fact and dimension tables optimized for analytical queries.
4. **Data Quality** — Validating consistency, integrity, and standardization at each layer.
**Source Systems:**
| System | Entities |
|---|---|
| CRM | Customer info, product info, sales details |
| ERP | Customer demographics, location, product category |
 
---
 
## ETL Process
 
The pipeline moves data through three stages using stored procedures:
 
| Step | Procedure | What it does |
|---|---|---|
| 1 | `bronze.load_bronze` | Truncates Bronze tables, then bulk-inserts raw data from the source CSV files. |
| 2 | `silver.load_silver` | Truncates Silver tables, then transforms and inserts cleansed data from Bronze — deduplicating customers, splitting product keys into category/product IDs, normalizing status/gender/country codes, fixing invalid dates, and recalculating sales figures where the source data is inconsistent. |
| 3 | Gold views | Query the Silver layer directly and join CRM + ERP data into ready-to-use dimension and fact views — no physical load step required. |
 
**Data lineage** (source → warehouse):
 
<img width="1094" height="554" alt="data_flow" src="https://github.com/zshafique25/Medallion-Data-Warehouse-SQL-Server/blob/master/docs/2_data_lineage.png" />
 
**CRM ↔ ERP integration logic:**
 
<img width="1522" height="761" alt="data_integration" src="https://github.com/zshafique25/Medallion-Data-Warehouse-SQL-Server/blob/master/docs/3_data_integration.png" />

---
 
## Data Model (Gold Layer)
 
The Gold layer exposes a **star schema** with two dimensions and one fact view:

<img width="1500" height="549" alt="data_model" src="https://github.com/zshafique25/Medallion-Data-Warehouse-SQL-Server/blob/master/docs/4_data_model.png" />
 
- **`gold.dim_customers`** — Customer demographics, enriched with gender/country from ERP where CRM data is missing.
- **`gold.dim_products`** — Current product catalog (historical/discontinued products filtered out), enriched with category and subcategory from ERP.
- **`gold.fact_sales`** — Sales transactions linked to both dimensions via surrogate keys, with order/shipping/due dates, quantity, price, and sales amount.
Full column-level documentation is available in [`docs/data_catalog.md`](docs/data_catalog.md).
 
---
 
## Data Quality Checks
 
The [`tests/`](tests) folder contains validation scripts run after each layer loads:
 
- **Silver checks** — null/duplicate primary keys, unwanted whitespace, and standardization of categorical fields (marital status, gender, country).
- **Gold checks** — surrogate key uniqueness in both dimensions, and referential integrity between `fact_sales` and its dimensions (no orphaned foreign keys).
---
 
## Tools & Technologies
 
- **SQL Server** — data warehouse engine
- **T-SQL** — DDL, stored procedures, and transformation logic
- **SSMS** — development and query execution

---
 
## Naming Conventions
 
All objects follow a consistent convention — see [`docs/naming_conventions.md`](docs/naming_conventions.md) for the full spec.
- `snake_case` throughout, English names, no reserved words.
- Bronze/Silver tables: `<sourcesystem>_<entity>` (e.g., `crm_cust_info`).
- Gold views: `<category>_<entity>` (e.g., `dim_customers`, `fact_sales`).
- Surrogate keys end in `_key`; system-generated columns are prefixed `dwh_`.
---

