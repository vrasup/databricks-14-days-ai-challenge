# DAY 6 (14/01/26) – Bronze → Silver → Gold Delta Lake Pipelines

## Learn

- Structuring Delta Lake pipelines with Bronze, Silver, and Gold layers
- Adding ingestion metadata to Bronze tables
- Cleaning and transforming Silver tables (nulls, outliers, deduplication)
- Deriving new columns like event_date and price_tier
- Aggregating metrics for Gold tables (product, category, daily KPIs)
- Calculating business metrics: views, purchases, revenue, conversion rates

## Tasks Completed

- Created Bronze table from raw CSV with ingestion timestamp
- Cleaned and transformed Silver layer
- Generated Gold tables at product, category, and daily levels
- Verified metrics and ensured consistency across layers
- Handled nulls, outliers, and duplicates properly
- Hands-On Practice

## Hands-On Practice
```python
# Step 0: Load raw CSV to Bronze
from pyspark.sql import functions as F

df = spark.read.csv(
    "/Volumes/workspace/ecommerce/ecommerce_data/2019-Nov.csv",
    header=True,
    inferSchema=True
)

df_bronze = df.withColumn("ingestion_ts", F.current_timestamp())

# Step 1: Save Bronze table
df_bronze.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("bronze_df_nov")
```
📌 Raw data stored with ingestion timestamp.
```python
# Step 2: Transform Silver layer
df_silver = df_bronze.dropna(subset=["event_time", "user_session", "product_id"]) \
                     .fillna({"brand":"unknown", "category_code":"unknown", "price":0}) \
                     .filter((F.col("price")>0) & (F.col("price")<10000)) \
                     .dropDuplicates(["user_session","event_time","product_id"]) \
                     .withColumn("event_date", F.to_date("event_time")) \
                     .withColumn("price_tier", 
                                 F.when(F.col("price")<10,"budget")
                                  .when(F.col("price")<50,"mid")
                                  .otherwise("premium")
                     )

# Step 3: Save Silver table
df_silver.write.format("delta") \
               .mode("overwrite") \
               .saveAsTable("silver_df_nov_realworld")
```
📌 Silver layer is cleaned, deduplicated, and enriched with derived columns.
```python
# Step 4: Aggregate Gold tables - Product Level
df_gold_product = df_silver.groupBy("product_id","category_id","category_code","brand") \
    .agg(
        F.countDistinct(F.when(F.col("event_type")=="view", F.col("user_id"))).alias("views"),
        F.countDistinct(F.when(F.col("event_type")=="purchase", F.col("user_id"))).alias("purchases"),
        F.sum(F.when(F.col("event_type")=="purchase", F.col("price"))).alias("revenue")
    ).withColumn(
        "conversion_rate",
        F.when(F.col("views")>0,F.col("purchases")/F.col("views")*100).otherwise(0)
    )

# Step 5: Save Gold Product table
df_gold_product.write.format("delta") \
                     .mode("overwrite") \
                     .saveAsTable("gold_product_metrics")
```
📌 Product-level KPIs computed: views, purchases, revenue, conversion rate.
```python
# Step 6: Aggregate Gold tables - Category Level
df_gold_category = df_silver.groupBy("category_id","category_code") \
    .agg(
        F.countDistinct(F.when(F.col("event_type")=="view", F.col("user_id"))).alias("views"),
        F.countDistinct(F.when(F.col("event_type")=="purchase", F.col("user_id"))).alias("purchases"),
        F.sum(F.when(F.col("event_type")=="purchase", F.col("price"))).alias("revenue")
    ).withColumn(
        "conversion_rate",
        F.when(F.col("views")>0,F.col("purchases")/F.col("views")*100).otherwise(0)
    )
```
📌 Category-level KPIs computed.
```python
# Step 7: Aggregate Gold tables - Daily Product Level
df_gold_daily = df_silver.groupBy("product_id","event_date") \
    .agg(
        F.countDistinct(F.when(F.col("event_type")=="view", F.col("user_id"))).alias("daily_views"),
        F.countDistinct(F.when(F.col("event_type")=="purchase", F.col("user_id"))).alias("daily_purchases"),
        F.sum(F.when(F.col("event_type")=="purchase", F.col("price"))).alias("daily_revenue")
    ).withColumn(
        "daily_conversion_rate",
        F.when(F.col("daily_views")>0,F.col("daily_purchases")/F.col("daily_views")*100).otherwise(0)
    )
```
📌 Daily product-level KPIs ready for analytics.

## **Output**

- **Bronze Table**
  - Raw e-commerce events stored as Delta
  - Ingestion timestamp added for lineage and auditability

- **Silver Table**
  - Cleaned and deduplicated data
  - Nulls handled for critical and non-critical columns
  - Outliers filtered and derived columns added
    - event_date
    - price_tier

- **Gold Tables**
  - Product-level KPIs: views, purchases, revenue, conversion rate
  - Category-level performance metrics
  - Daily product-level time series metrics


## **Key Takeaways**

- Bronze layer preserves raw ingested data
- Silver layer applies cleaning, deduplication, and enrichment
- Gold layer provides aggregated KPIs for analytics
- Derived columns like event_date and price_tier enable better insights
- Handling nulls, outliers, and duplicates is critical for reliable analytics

## **Resources**

[Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)

[Architecture Video](https://youtu.be/njjBdmAQnR0?si=prwVGQCPZj-ZkbGw)

[Build a Medallion Architecture with Databricks](https://youtu.be/yy9H4mlOG6I?si=WtaiQKm1kyROqNnh)

[Incremental processing patterns](https://youtu.be/GjV2m8b9fNY?si=IAp558OgaGN7NrSP)


## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC
