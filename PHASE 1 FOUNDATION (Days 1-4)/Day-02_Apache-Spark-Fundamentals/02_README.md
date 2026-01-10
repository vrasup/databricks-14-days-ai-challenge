# **DAY 2 (10/01/26) – Apache Spark Fundamentals**

## **Learn**
- Spark architecture (driver, executors, DAG)
- DataFrames vs RDDs
- Lazy evaluation
- Notebook magic commands (`%sql`, `%python`, `%fs`)

## **Tasks Completed**
1. Uploaded sample e-commerce CSV
2. Read data into DataFrame
3. Performed basic operations: select, filter, groupBy, orderBy
4. Exported results

## **Hands-On Practice**
```python
# Load Data
df = spark.read.csv("/path/to/ecommerce_data.csv", header=True, inferSchema=True)
df.show(5)

# Basic operations
df.select("event_type", "product_id", "price").show(10)
df.filter("price > 100").show(10)
df.groupBy("event_type").count().show()
top_brands = df.groupBy("brand").count().orderBy("count", ascending=False).limit(5)
top_brands.show()
```
## **Output**
```
+----------+----------+-------+
|event_type|product_id|  price|
+----------+----------+-------+
|      view|  44600062|  35.79|
|      view|   3900821|   33.2|
|      view|  17200506|  543.1|
|      view|   1307067| 251.74|
|      view|   1004237|1081.98|
|      view|   1480613| 908.62|
|      view|  17300353| 380.96|
|      view|  31500053|  41.16|
|      view|  28719074| 102.71|
|      view|   1004545| 566.01|
+----------+----------+-------+
only showing top 10 rows
```
## **Key Takeaways**

- Spark DataFrames provide a structured, distributed way to process big data

- Lazy evaluation helps optimize computation

- GroupBy, filter, and orderBy are core operations for analysis

## **Resources**

[PySpark Guide](https://www.databricks.com)

[Spark SQL Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC