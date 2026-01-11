# DAY 3 (11/01/26) – PySpark Transformations Deep Dive

## Learn
- PySpark vs Pandas
- Joins (inner, left, right, outer)
- Window functions (running totals, rankings)
- Derived feature creation

## Tasks Completed
1. Loaded full e-commerce dataset into Spark DataFrame
2. Explored schema, null values, and categorical distributions
3. Performed analytical joins on event data
4. Implemented window functions for running totals
5. Ranked products by revenue
6. Calculated conversion rates by category

## Hands-On Practice
```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Filter purchase events
purchases = df.filter(F.col("event_type") == "purchase")

# Revenue per product
revenue = purchases.groupBy("product_id", "category_code") \
    .agg(F.sum("price").alias("revenue"))

# Rank products by revenue within category
rank_window = Window.partitionBy("category_code").orderBy(F.desc("revenue"))
top_products = revenue.withColumn("rank", F.rank().over(rank_window))
top_products.show(10)

# Running total per user
user_window = Window.partitionBy("user_id").orderBy("event_time")

df_running = df.withColumn(
    "cumulative_events",
    F.count("*").over(user_window)
)
df_running.show(10)

# Conversion rate by category
conversion = df.groupBy("category_code", "event_type").count() \
    .pivot("event_type").sum("count") \
    .withColumn(
        "conversion_rate",
        (F.col("purchase") / F.col("view")) * 100
    )
conversion.show(5)
```
## **Output**

```
## **Top Products by Revenue**
+----------+---------------+------------------+----+
|product_id|category_code  |revenue           |rank|
+----------+---------------+------------------+----+
|28401058  |accessories.bag|1505.85           |1   |
|21000006  |accessories.bag|1194.28           |2   |
|28400775  |accessories.bag|861.28            |3   |
+----------+---------------+------------------+----+
```
## **Running Total per User**
```
+-------------------+---------+-----------------+
|event_time         |user_id  |cumulative_events|
+-------------------+---------+-----------------+
|2019-10-09 10:30:19|205053188|1                |
|2019-10-09 10:30:44|205053188|2                |
+-------------------+---------+-----------------+
```
## **Conversion Rate by Category**
```
+--------------------+------+--------+------------------+
|category_code       |view  |purchase|conversion_rate  |
+--------------------+------+--------+------------------+
|stationery.cartrige |7380  |134     |1.81              |
+--------------------+------+--------+------------------+
```
## **Key Takeaways**

- Window functions enable analytical calculations without collapsing rows
- `partitionBy` defines the grouping scope for window operations
- `orderBy` controls the sequence of calculations within each partition
- Running totals and rankings are common analytical patterns
- PySpark is well suited for large-scale distributed analytics

## **Resources**

[Window Functions](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/window.html)

[PySpark Functions API](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC