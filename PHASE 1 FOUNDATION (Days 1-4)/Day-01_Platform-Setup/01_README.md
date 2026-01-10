 # **DAY 1 (09/01/26) – Platform Setup & First Steps**

## **Learn**
- Why Databricks vs Pandas/Hadoop
- Lakehouse architecture basics
- Databricks workspace structure
- Industry use cases (Netflix, Shell, Comcast)

## **Tasks Completed**
1. Create Databricks Community Edition account
2. Navigate Workspace, Compute, Data Explorer
3. Create first notebook
4. Run basic PySpark commands

## **Hands-On Practice**
```python
# Create simple DataFrame
data = [("iPhone", 999), ("Samsung", 799), ("MacBook", 1299)]
df = spark.createDataFrame(data, ["product", "price"])
df.show()

# Filter expensive products
df.filter(df.price > 1000).show()
```
## **Output:**
```
+-------+-----+
|product|price|
+-------+-----+
| iPhone|  999|
|Samsung|  799|
|MacBook| 1299|
+-------+-----+
```
## **Filtered (price > 1000):**
```
+-------+-----+
|product|price|
+-------+-----+
|MacBook| 1299|
+-------+-----+
```
## **Key Takeaways**

- Databricks provides an integrated Lakehouse platform combining data engineering and analytics.
- PySpark enables scalable data transformations using lazy evaluation.
- Databricks notebooks offer an interactive and efficient development environment.

## **Resources**
[Databricks](https://www.databricks.com)

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC