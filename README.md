# E-Commerce Advanced SQL Analytics: Cohort Retention & RFM Segmentation 📊

This repository features a comprehensive data analytics project focused on computing critical business metrics, analyzing user retention via cohorts, and performing customer segmentation. The analysis is implemented directly using complex **SQL queries (Window Functions, CTEs, and Conditional Aggregations)**.

### 📐 Project Objectives & Solved Tasks

1. **Descriptive & Geodemographic Analytics:** Evaluated sales dynamics, total order volumes, and Average Order Value (AOV) across different countries.
2. **Growth Metrics Tracking:** Designed window functions to track cumulative revenue growth (Running Total) over consecutive months.
3. **Cohort Retention Analysis:** Developed a comprehensive query to track user behavior over time, determining the percentage of returning customers (*Retention Rate %*) across monthly cohorts.
4. **RFM Customer Segmentation:** Classified customer base using `NTILE` window functions to assign behavioral ranks based on Recency, Frequency, and Monetary metrics.

### 🛠️ Tech Stack & Tools
* **SQL Dialect:** SQLite (utilizing CTEs, Window Functions, and Date Manipulations)
* **Environment:** Jupyter Notebook, Python 3 (Pandas & SQLite3 core libraries)
* **Data Output:** Multi-sheet analytical tables exported to Microsoft Excel for dashboard connection

---

### 🧠 Featured SQL Code Examples

#### 1. Cohort Retention Rate (%) Matrix
*Demonstrates dynamic calculation of user lifespan periods and percentage distributions:*
```sql
WITH user_cohorts AS (
    SELECT user_id, MIN(STRFTIME('%Y-%m', order_date)) AS cohort_month
    FROM orders GROUP BY user_id
),
order_periods AS (
    SELECT o.user_id, c.cohort_month,
        (CAST(SUBSTR(STRFTIME('%Y-%m', o.order_date), 1, 4) AS INT) - CAST(SUBSTR(c.cohort_month, 1, 4) AS INT)) * 12 +
        (CAST(SUBSTR(STRFTIME('%Y-%m', o.order_date), 6, 2) AS INT) - CAST(SUBSTR(c.cohort_month, 6, 2) AS INT)) AS period_month
    FROM orders o JOIN user_cohorts c ON o.user_id = c.user_id
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(DISTINCT CASE WHEN period_month = 0 THEN user_id END) AS cohort_size
    FROM order_periods GROUP BY cohort_month
)
SELECT p.cohort_month, s.cohort_size,
    ROUND(COUNT(DISTINCT CASE WHEN p.period_month = 0 THEN p.user_id END) * 100.0 / s.cohort_size, 2) || '%' AS month_0_pct,
    ROUND(COUNT(DISTINCT CASE WHEN p.period_month = 1 THEN p.user_id END) * 100.0 / s.cohort_size, 2) || '%' AS month_1_pct,
    ROUND(COUNT(DISTINCT CASE WHEN p.period_month = 2 THEN p.user_id END) * 100.0 / s.cohort_size, 2) || '%' AS month_2_pct
FROM order_periods p JOIN cohort_sizes s ON p.cohort_month = s.cohort_month
GROUP BY p.cohort_month ORDER BY p.cohort_month;
```

#### 2. RFM Segmentation Query
*Demonstrates customer filtering using mathematical date conversions and mathematical partition bucketing:*
```sql
WITH user_rfm_raw AS (
    SELECT user_id,
        CAST(JULIANDAY('2026-03-31') - JULIANDAY(MAX(order_date)) AS INT) AS recency_days,
        COUNT(order_id) AS frequency_count, SUM(total_amount) AS monetary_value
    FROM orders GROUP BY user_id
),
rfm_scores AS (
    SELECT user_id, recency_days, frequency_count, monetary_value,
        NTILE(3) OVER (ORDER BY recency_days DESC) AS r_score,
        NTILE(3) OVER (ORDER BY frequency_count ASC) AS f_score,
        NTILE(3) OVER (ORDER BY monetary_value ASC) AS m_score
    FROM user_rfm_raw
)
SELECT user_id, recency_days, frequency_count, ROUND(monetary_value, 2) AS monetary_value,
    (r_score || f_score || m_score) AS rfm_segment
FROM rfm_scores ORDER BY rfm_segment DESC, monetary_value DESC;
```
