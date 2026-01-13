# DAY 5 (13/01/26) – Delta Lake Advanced (MERGE, Time Travel, Optimize & Vacuum)

## Learn

- Incremental data processing using MERGE (UPSERT)
- How Delta Lake maintains table versions
- Time Travel using version and timestamp
- Performance optimization with OPTIMIZE & ZORDER
- Safe cleanup of old data files using VACUUM

## Tasks Completed

- Simulated incremental data updates on an existing Delta table
- Performed UPSERT operations using Delta MERGE
- Verified table version history using DESCRIBE HISTORY
- Queried historical versions of the Delta table (Time Travel)
- Optimized table layout using OPTIMIZE and ZORDER
- Cleaned unused data files safely using VACUUM

## Hands-On Practice
```python
#Step 1: Load existing Delta table
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "ecommerce_oct2019_sql")

#Step 2: Simulate incremental data
from pyspark.sql import functions as F

updates_df = df.limit(10) \
    .withColumn("price", F.col("price") + 100)
```
📌 This simulates today’s incoming data where prices changed for some records.
```python
#Step 3: Incremental MERGE (UPSERT)
delta_table.alias("t").merge(
    updates_df.alias("s"),
    "t.event_time = s.event_time AND t.user_id = s.user_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```
📌 Existing records are updated, new records are inserted, and unchanged records remain untouched.
```python
#Step 4: Check Delta table history
spark.sql("DESCRIBE HISTORY ecommerce_oct2019_sql").show()

#Step 5: Time Travel (read old version)
spark.read.format("delta") \
    .option("versionAsOf", 0) \
    .table("ecommerce_oct2019_sql") \
    .show(5)
```
📌 Reads the table before the MERGE operation.
```python
#Step 6: Optimize table performance
spark.sql("""
OPTIMIZE ecommerce_oct2019_sql
ZORDER BY (event_type, user_id)
""")

#Step 7: Clean unused files safely
spark.sql("""
VACUUM ecommerce_oct2019_sql RETAIN 168 HOURS
""")
```
📌 Deletes unused old files

📌 Keeps last 7 days of history for rollback safety

## **Output**
```
Delta History Snapshot
version | operation | timestamp
0       | CREATE    | table created
1       | MERGE     | incremental update
2       | OPTIMIZE  | file compaction
3       | VACUUM    | cleanup start
4       | VACUUM    | cleanup completed
```
## **Key Takeaways**

- MERGE enables efficient incremental updates (UPSERT)
- Delta Lake stores every change as a new version
- Time Travel allows querying data before and after updates
- OPTIMIZE improves performance without changing data
- VACUUM removes unused files but keeps recent versions safe
- Delta Lake ensures data is never accidentally lost

## **Resources**

[Time Travel](https://www.databricks.com/blog/2019/02/04/introducing-delta-time-travel-for-large-scale-data-lakes.html)

[MERGE Guide](https://docs.databricks.com/aws/en/delta/merge)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC

