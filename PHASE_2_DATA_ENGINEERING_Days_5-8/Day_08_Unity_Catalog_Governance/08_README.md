

# DAY 8 (16/01/26) – Unity Catalog & Data Governance

## Learn

* Creating catalogs and schemas in Databricks Unity Catalog
* Registering Delta tables under governed schemas
* Implementing controlled access using views
* Handling incremental data access in a secure, governed way
* Understanding the difference between managed tables and views for access control
* Documenting lineage and verifying tables and views

## Tasks Completed

* Created `ecommerce` catalog with `bronze`, `silver`, and `gold` schemas
* Registered Delta tables in each schema (Bronze, Silver, Gold)
* Conceptually set up permissions using governed views (Free Edition limitation)
* Created Gold-layer views for controlled access (`top_products`)
* Verified table and view lineage and schema details

## Hands-On Practice

```python
# Step 0: Create Catalog & Schemas (SQL)
spark.sql("CREATE CATALOG IF NOT EXISTS ecommerce")
spark.sql("CREATE SCHEMA IF NOT EXISTS ecommerce.bronze")
spark.sql("CREATE SCHEMA IF NOT EXISTS ecommerce.silver")
spark.sql("CREATE SCHEMA IF NOT EXISTS ecommerce.gold")

# Step 1: Register Delta Tables
# Bronze table
spark.sql("""
CREATE TABLE IF NOT EXISTS ecommerce.bronze.bronze_events
USING DELTA
LOCATION '/user/hive/warehouse/bronze_df_nov'
""")

# Silver table
spark.sql("""
CREATE TABLE IF NOT EXISTS ecommerce.silver.silver_events
USING DELTA
LOCATION '/user/hive/warehouse/silver_df_nov_realworld'
""")

# Gold tables
spark.sql("""
CREATE TABLE IF NOT EXISTS ecommerce.gold.product_metrics
USING DELTA
LOCATION '/user/hive/warehouse/gold_df_product_nov'
""")
spark.sql("""
CREATE TABLE IF NOT EXISTS ecommerce.gold.category_metrics
USING DELTA
LOCATION '/user/hive/warehouse/gold_df_category_nov'
""")
spark.sql("""
CREATE TABLE IF NOT EXISTS ecommerce.gold.daily_metrics
USING DELTA
LOCATION '/user/hive/warehouse/gold_df_daily_nov'
""")

# Step 2: Create Controlled Access Views
spark.sql("""
CREATE OR REPLACE VIEW ecommerce.gold.top_products AS
SELECT product_id, brand, revenue, conversion_rate
FROM ecommerce.gold.product_metrics
WHERE revenue > 0
ORDER BY revenue DESC
LIMIT 100
""")

# Step 3: Verify
spark.table("ecommerce.gold.top_products").show(5)
```

📌 Registered Delta tables under Unity Catalog schemas with a controlled access view for Gold metrics.

---

## **Output**

* **Bronze Table**

  * Raw CSV ingested with ingestion timestamp and date

* **Silver Table**

  * Cleaned, deduplicated, and enriched with derived columns
  * Incremental processing supported

* **Gold Tables**

  * Product, category, and daily metrics aggregated
  * Conversion rates and revenue KPIs calculated
  * `top_products` view created for controlled access

---

## **Key Takeaways**

* Unity Catalog allows catalogs and schemas for governance
* Delta tables registered under governed schemas are discoverable and queryable
* Views provide controlled access, even in Free Edition where GRANT/REVOKE isn’t supported
* Documentation and lineage verification ensure reliable data governance
* Incremental updates flow seamlessly from Bronze → Silver → Gold

---

## **Resources**

[Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog)

[Getting Started](https://docs.databricks.com/aws/en/data-governance/unity-catalog/get-started)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC 

---

