# DAY 10 (18/01/26) – Performance Optimization

## Learn

* Reading Spark query execution plans (Parsed, Analyzed, Optimized, Physical)
* Partitioning large Delta tables for efficient querying
* Using ZORDER clustering for faster lookups on high-cardinality columns
* Benchmarking query performance before and after optimization
* Understanding Serverless compute limitations (e.g., caching not supported)

---

## Tasks Completed

* Analyzed execution plans for Silver-layer events
* Partitioned Silver events table by `event_date` and `event_type`
* Applied ZORDER on `user_id` and `product_id` for optimized filtering
* Benchmarked queries to measure performance improvement
* Learned Serverless-specific constraints for caching and iterative queries

---

## Hands-On Practice

```python
# Step 0: Combine October + November Silver tables (if not done already)
spark.sql("""
CREATE OR REPLACE TABLE default.silver_events_all AS
SELECT * FROM default.silver_events_oct
UNION ALL
SELECT * FROM default.silver_df_nov_realworld
""")

# Step 1: Explain Query Plan
spark.sql("SELECT * FROM default.silver_events_all WHERE event_type='purchase'").explain(True)

# Step 2: Partition Large Table
spark.sql("""
CREATE OR REPLACE TABLE default.silver_events_all_part
USING DELTA
PARTITIONED BY (event_date, event_type)
AS SELECT * FROM default.silver_events_all
""")

# Step 3: Apply ZORDER for Optimized Lookups
spark.sql("OPTIMIZE default.silver_events_all_part ZORDER BY (user_id, product_id)")

# Step 4: Benchmark Performance (Before vs After)

import time

# Before Optimization
start = time.time()
spark.sql("""
SELECT *
FROM default.silver_events_all
WHERE event_type = 'purchase'
AND event_date = '2019-11-10'
""").count()
print(f"Unoptimized time: {time.time() - start:.2f} seconds")

# After Optimization
start = time.time()
spark.sql("""
SELECT *
FROM default.silver_events_all_part
WHERE event_type = 'purchase'
AND event_date = '2019-11-10'
""").count()
print(f"Optimized time: {time.time() - start:.2f} seconds")

# Step 5: Note - Caching is NOT supported on Serverless compute
# cached_df = spark.table("default.silver_events_all_part").cache()
# cached_df.count()  # This will fail on Serverless
```

---

## **Output**

* **Query Plan Analysis**

  * Parsed, Analyzed, Optimized, Physical plans clearly show filters and projections
  * Photon engine used automatically in Serverless compute for faster execution

* **Partitioned Table**

  * `silver_events_all_part` partitioned by `event_date` and `event_type`
  * Reduced unnecessary scans for queries filtering by these columns

* **ZORDER Optimization**

  * Data clustered by `user_id` and `product_id` to speed up lookups on high-cardinality columns

* **Benchmark**

  * Before Optimization: ~0.86 seconds
  * After Optimization: ~1.41 seconds (depends on query type; some filters benefit more)

* **Caching**

  * Not supported on Serverless compute; rely on partitioning + ZORDER for performance

---

## **Key Takeaways**

* Reading Spark query plans is critical to understanding how your queries execute
* Partitioning reduces I/O and speeds up filters on known high-cardinality columns
* ZORDER clustering optimizes table layout for repeated, selective queries
* Serverless compute uses Photon engine to auto-optimize queries, but caching is limited
* Benchmarking is essential to quantify improvements and validate optimization choices

---

## **Resources**

* [Databricks Delta Optimization Guide](https://docs.databricks.com/delta/optimizations/index.html)

* [Spark SQL and DataFrames Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

* [Photon Engine](https://docs.databricks.com/runtime/photon/index.html)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC 

---

