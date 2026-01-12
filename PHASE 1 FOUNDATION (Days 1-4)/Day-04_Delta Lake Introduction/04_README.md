# DAY 4 (12/01/26) – Delta Lake Introduction

## Learn
- What is Delta Lake and why it is used
- ACID transactions in data lakes
- Schema enforcement and schema evolution
- Delta Lake vs Parquet

## Tasks Completed
1. Read a large CSV dataset from Databricks Volume into Spark DataFrame
2. Converted raw CSV data into Delta format
3. Created managed Delta tables using PySpark and SQL
4. Verified Delta table metadata and storage format
5. Tested schema enforcement by attempting invalid inserts
6. Understood how Delta prevents bad writes and ensures data quality

## Hands-On Practice
```python
# Step 1: Read CSV from Databricks Volume
path = "/Volumes/workspace/ecommerce/ecommerce_data/2019-Oct.csv"

df = spark.read.csv(
    path,
    header=True,
    inferSchema=True
)

df.printSchema()
df.show(5)

# Step 2: Write DataFrame as a Delta table (Managed Table)
df.write.format("delta").mode("overwrite").saveAsTable("ecommerce_oct2019")
```
## Step 3: Verify Delta table
```sql
SHOW TABLES;

SELECT * 
FROM ecommerce_oct2019
LIMIT 5;
```
## Step 4: Test schema enforcement
```python
wrong_schema_df = spark.createDataFrame(
    [(1, "test", 100)],
    ["id", "name", "value"]
)

try:
    wrong_schema_df.write.format("delta") \
        .mode("append") \
        .saveAsTable("ecommerce_oct2019")
except Exception as e:
    print("Schema enforcement triggered!")
    print(e)
```
## **Output**
```
## **Delta Table Preview**
+-------------------+----------+----------+-------------------+--------------------+---------+-------+---------+--------------------+
|event_time         |event_type|product_id|category_id        |category_code       |brand    |price  |user_id  |user_session        |
+-------------------+----------+----------+-------------------+--------------------+---------+-------+---------+--------------------+
|2019-10-12 11:58:42|view      |1005110   |2053013555631882655|electronics.smart...|apple    |1092.68|547801997|d0bc9270-888c-4dd...|
+-------------------+----------+----------+-------------------+--------------------+---------+-------+---------+--------------------+
```
## **Key Takeaways**

- Delta Lake adds ACID transactions on top of Parquet
- Delta enforces schema to prevent corrupt or invalid writes
- Managed Delta tables simplify data governance
- Delta tables store metadata in a transaction log (_delta_log)
- Delta is the default and recommended table format in Databricks

## **Resources**

[Delta Lake Docs](https://docs.databricks.com/aws/en/delta)

[Delta Tutorial](https://docs.databricks.com/aws/en/delta/tutorial)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC