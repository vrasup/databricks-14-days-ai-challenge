# DAY 7 (15/01/26) – Workflows & Job Orchestration

## Learn

- Adding parameter widgets to notebooks for dynamic inputs
- Creating multi-task Jobs in Databricks linking Bronze → Silver → Gold
- Setting dependencies between tasks to ensure proper execution order
- Scheduling jobs to automatically run daily
- Handling incremental processing for new data in Bronze
- Error handling and verification for reliable pipelines

## Tasks Completed

- Added widgets for source path, table names, and processing date
- Created a multi-task Job connecting Bronze, Silver, and Gold notebooks
- Configured task dependencies: Silver depends on Bronze, Gold depends on Silver
- Tested the pipeline with manual runs to verify end-to-end success
- Scheduled job to run automatically at 2:00 AM daily
- Verified that new raw data flows correctly through all layers

## Hands-On Practice
```python
# Step 0: Add widgets for parameters
dbutils.widgets.text("source_path","/default/path")
dbutils.widgets.text("bronze_table","bronze_events")
dbutils.widgets.text("silver_table","silver_events")
dbutils.widgets.text("gold_table","gold_events")
dbutils.widgets.text("processing_date","2026-01-15")

# Step 1: Read parameters
source_path = dbutils.widgets.get("source_path")
bronze_table = dbutils.widgets.get("bronze_table")
silver_table = dbutils.widgets.get("silver_table")
gold_table = dbutils.widgets.get("gold_table")
processing_date = dbutils.widgets.get("processing_date")

# Step 2: Bronze ingestion
df_bronze = spark.read.csv(source_path, header=True, inferSchema=True) \
                     .withColumn("ingestion_ts", F.current_timestamp()) \
                     .withColumn("ingestion_date", F.to_date("ingestion_ts"))
df_bronze.write.format("delta").mode("append").partitionBy("ingestion_date").saveAsTable(bronze_table)

# Step 3: Silver transformation (incremental)
df_silver = spark.table(bronze_table) \
                  .filter(F.to_date("ingestion_ts") == processing_date) \
                  .dropDuplicates(["user_session","event_time","product_id"]) \
                  .withColumn("event_date", F.to_date("event_time")) \
                  .withColumn("price_tier", F.when(F.col("price")<10,"budget")
                                            .when(F.col("price")<50,"mid")
                                            .otherwise("premium"))
df_silver.write.format("delta").mode("append").option("mergeSchema","true").saveAsTable(silver_table)

# Step 4: Gold aggregates
df_gold_product = df_silver.groupBy("product_id","category_id","category_code","brand") \
    .agg(F.countDistinct(F.when(F.col("event_type")=="view",F.col("user_id"))).alias("views"),
         F.countDistinct(F.when(F.col("event_type")=="purchase",F.col("user_id"))).alias("purchases"),
         F.sum(F.when(F.col("event_type")=="purchase",F.col("price"))).alias("revenue")) \
    .withColumn("conversion_rate", F.when(F.col("views")>0,F.col("purchases")/F.col("views")*100).otherwise(0))
df_gold_product.write.format("delta").mode("append").option("mergeSchema","true").saveAsTable(gold_table + "_product")
```
📌 End-to-end pipeline with parameters, incremental filtering, and Delta writes.

## **Output**

- **Bronze Table**

   - Raw CSV ingested with ingestion timestamp and date

- **Silver Table**

   - Incremental data filtered by processing date

   - Cleaned, deduplicated, and enriched with derived columns

- **Gold Tables**

  - Aggregated metrics at product, category, and daily levels

  - Conversion rates and business KPIs calculated

## **Key Takeaways**

- Jobs UI allows orchestration of multiple notebooks
- Dependencies ensure tasks run in the correct order
- Parameters make notebooks dynamic and reusable
- Scheduling triggers daily automation for new data
- Incremental processing ensures efficient updates without full reloads

## **Resources**
[Jobs Documentation](https://docs.databricks.com/aws/en/jobs)

[Job Parameters](https://docs.databricks.com/aws/en/jobs/parameters)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC
