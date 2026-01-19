
# DAY 11 (19/01/26) – Statistical Analysis & ML Feature Engineering

## Learn

* Computing **descriptive statistics** in Spark (`count`, `mean`, `stddev`, `min`, `max`)
* Understanding **median and quartiles** using `approxQuantile()`
* Performing **hypothesis testing** (weekend vs weekday conversion rates)
* Calculating **correlations** between numeric features (e.g., price vs conversion rate)
* Engineering ML-ready features:

  * Log transformation of skewed numeric features (`price_log`)
  * Time-based features (`hour`, `day_of_week`, `is_weekend`)
  * User behavioral features (`time_since_first_event`) using **window functions**
  * Encoding binary labels for ML (`purchase = 1, else 0`)

---

## Tasks Completed

* Filtered out **price anomalies** and applied **log transformation**
* Generated **time-based features** for ML (hour, day, weekend flag)
* Built **user-level behavioral features** with Spark window functions
* Calculated **conversion rates** for weekdays vs weekends and validated hypothesis
* Created **product-level correlations** between price and conversion rate
* Selected **final ML feature set** for modeling

---

## Hands-On Practice

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Step 0: Filter anomalies
events_fe = events.filter(F.col("price") > 0)

# Step 1: Log transformation
events_fe = events_fe.withColumn("price_log", F.log(F.col("price")))

# Step 2: Time-based features
events_fe = events_fe.withColumn("hour", F.hour("event_time"))
events_fe = events_fe.withColumn("day_of_week", F.dayofweek("event_date"))
events_fe = events_fe.withColumn("is_weekend", F.dayofweek("event_date").isin([1,7]).cast("int"))

# Step 3: User behavioral feature
user_window = Window.partitionBy("user_id").orderBy("event_time")
events_fe = events_fe.withColumn(
    "time_since_first_event",
    F.col("event_time").cast("long") - F.first(F.col("event_time")).over(user_window).cast("long")
)

# Step 4: Encode label
events_fe = events_fe.withColumn(
    "label",
    F.when(F.col("event_type") == "purchase", 1).otherwise(0)
)

# Step 5: Select final ML features
ml_features = events_fe.select(
    "price_log",
    "hour",
    "day_of_week",
    "is_weekend",
    "time_since_first_event",
    "label"
)

ml_features.show()
ml_features.printSchema()
```

---

## **Output**

* **Descriptive Stats**

  * Count: 109,585,110
  * Mean price: 292.16
  * Median: 164.48
  * 25% quantile: 67.87, 75% quantile: 360.09

* **Conversion Rate Analysis**

  * Weekend: 1.80%
  * Weekday: 1.49%
  * Null hypothesis rejected → higher conversion on weekends

* **Correlation Analysis**

  * Correlation between average price and conversion rate: -0.043 → weak negative correlation

* **ML Feature Set**

  * `price_log`, `hour`, `day_of_week`, `is_weekend`, `time_since_first_event`, `label`
  * Ready for ML modeling

---

## **Key Takeaways**

* Median and quartiles help understand skewed distributions better than mean/std
* Weekday vs weekend analysis validates **behavioral patterns** in e-commerce
* Correlation analysis identifies weak or strong predictors
* Spark **window functions** are powerful for user-level behavioral metrics
* Feature engineering transforms raw event logs into ML-ready datasets

---

## **Resources**

* [Spark ML Guide](https://spark.apache.org/docs/latest/ml-guide.html)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC 

---

