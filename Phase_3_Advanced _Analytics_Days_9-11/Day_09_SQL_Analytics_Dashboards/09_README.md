
# DAY 9 (17/01/26) – SQL Analytics & Dashboards

## Learn

* Using Databricks SQL with Serverless Starter Warehouse (Free Edition)
* Writing analytical SQL queries on Gold-layer data
* Funnel analysis and conversion metrics
* Building interactive visualizations
* Creating dashboards with multiple charts
* Understanding aggregation choices for KPIs

---

## Tasks Completed

* Used Serverless Starter SQL Warehouse for analytics
* Wrote complex analytical SQL queries on Gold tables
* Built revenue trend, funnel, and top-product analyses
* Created visualizations (line chart, funnel chart, bar chart)
* Combined queries into a Databricks SQL Dashboard

---

## Hands-On Practice

```sql
-- Query 1: Revenue Trends Over Time
SELECT
  event_date,
  COALESCE(SUM(daily_revenue), 0) AS total_revenue,
  COALESCE(SUM(daily_purchases), 0) AS total_purchases,
  COALESCE(SUM(daily_views), 0) AS total_views,
  CASE
    WHEN SUM(daily_views) > 0
    THEN ROUND(SUM(daily_purchases) * 100.0 / SUM(daily_views), 2)
    ELSE 0
  END AS conversion_rate
FROM gold_daily_metrics_all
GROUP BY event_date
ORDER BY event_date;
```

```sql
-- Query 2: Funnel Analysis (View → Cart → Purchase)

-- Step 0: Combine October + November Silver tables
WITH combined_silver AS (
    SELECT event_date, user_id, event_type FROM default.silver_events_oct
    UNION ALL
    SELECT event_date, user_id, event_type FROM default.silver_df_nov_realworld
),

-- Step 1: Aggregate by event type to get unique users per stage
funnel_data AS (
    SELECT
        event_type AS step,
        COUNT(DISTINCT user_id) AS value
    FROM combined_silver
    GROUP BY event_type
)

-- Step 2: Optional conversion percentage
SELECT
    step,
    value,
    ROUND(
        CASE 
            WHEN step = 'purchase' THEN 
                 100.0 * value / (SELECT value FROM funnel_data WHERE step='view')
            ELSE NULL
        END, 2
    ) AS conversion_rate
FROM funnel_data
ORDER BY 
    CASE step
        WHEN 'view' THEN 1
        WHEN 'cart' THEN 2
        WHEN 'purchase' THEN 3
    END;

```

```sql
-- Query 3: Top Products by Revenue
-- Top Products Combined (Oct + Nov)
SELECT
  product_id,
  brand,
  SUM(revenue) AS total_revenue,
  AVG(conversion_rate) AS avg_conversion_rate
FROM gold_product_metrics_all
GROUP BY product_id, brand
ORDER BY total_revenue DESC
LIMIT 10;
```

📌 All queries were executed using the **Databricks SQL Editor** and visualized using built-in chart options.

---

## **Output**

* **Revenue Trend Chart**

  * Line chart showing daily revenue and conversion patterns

* **Funnel Analysis**

  * Funnel visualization highlighting user drop-off from view → purchase

* **Top Products Chart**

  * Bar chart showing highest revenue-generating products

* **Dashboard**

  * Combined all charts into a single SQL Dashboard for business insights

---

## **Key Takeaways**

* SQL Warehouses enable analytics without managing clusters
* Gold-layer tables are optimized for reporting and dashboards
* Choosing correct aggregations (SUM vs AVG) is critical for KPIs
* Funnel charts clearly highlight conversion bottlenecks
* Dashboards turn raw metrics into decision-ready insights

---

## **Resources**

[Databricks SQL](https://docs.databricks.com/aws/en/sql)

[Dashboards Guide](https://docs.databricks.com/aws/en/dashboards)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC

---

