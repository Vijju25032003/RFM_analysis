# RFM_analysis
🚀 RFM Customer Segmentation using Weighted RFM Z-Score

Completed an end-to-end customer segmentation analytics project on the Olist E-Commerce Dataset, where I designed a robust RFM (Recency, Frequency, Monetary) model using SQL, enhanced with Z-score normalization and weighted scoring to generate actionable business insights.

🔍 Project Highlights

Built RFM metrics using optimized SQL Views & CTEs

Applied Z-score normalization to reduce skewness in Recency, Frequency, and Monetary values

Designed a Weighted RFM Score:

Recency: 40%

Frequency: 30%

Monetary: 30%

Segmented customers into:

🏆 Champions

💎 Loyal Customers

🌱 Potential Loyalists

⚠️ At-Risk

❌ Lost Customers

Identified high-value revenue drivers and churn-risk customer groups

📈 Business Impact

This data-driven segmentation enables businesses to:

Improve customer retention strategies

Run targeted marketing campaigns

Maximize customer lifetime value (CLV)

Reduce churn with proactive insights

🛠️ Tech Stack

SQL | CTEs | Views | Z-Score Normalization | Customer Analytics | RFM Modeling



WITH completed_orders AS (
    SELECT 
        o.order_id,
        c.customer_unique_id,
        DATEDIFF(CURRENT_DATE, o.order_purchase_timestamp) AS recency
    FROM orders o
    JOIN customers c 
        ON o.customer_id = c.customer_id
    WHERE o.order_status = 'delivered'
),

monetary_value AS (
    SELECT 
        oi.order_id,
        SUM(oi.price) AS order_amount
    FROM order_items oi
    GROUP BY oi.order_id
),

rfm_raw AS (
    SELECT
        c.customer_unique_id,
        MIN(c.recency) AS recency,
        COUNT(c.order_id) AS frequency,
        SUM(m.order_amount) AS monetary
    FROM completed_orders c
    LEFT JOIN monetary_value m 
        ON c.order_id = m.order_id
    GROUP BY c.customer_unique_id
),

stats AS (
    SELECT
        AVG(recency) AS avg_r,
        STDDEV_POP(recency) AS std_r,
        AVG(frequency) AS avg_f,
        STDDEV_POP(frequency) AS std_f,
        AVG(monetary) AS avg_m,
        STDDEV_POP(monetary) AS std_m
    FROM rfm_raw
),

rfm_z AS (
    SELECT
        r.customer_unique_id,
        r.recency,
        r.frequency,
        r.monetary,

        -- ⭐ Z-scores (standardized)
        -- Recency is reversed: lower recency = better → multiply by -1
        -(r.recency - s.avg_r) / NULLIF(s.std_r, 0) AS r_z,
        (r.frequency - s.avg_f) / NULLIF(s.std_f, 0) AS f_z,
        (r.monetary - s.avg_m) / NULLIF(s.std_m, 0) AS m_z
    FROM rfm_raw r
    CROSS JOIN stats s
),

rfm_final AS (
    SELECT
        *,
        -- ⭐ Weighted score (you can adjust)
        (r_z * 0.4) + (f_z * 0.3) + (m_z * 0.3) AS rfm_score
    FROM rfm_z
)

SELECT *,
       CASE
            WHEN rfm_score >= 1.0 THEN 'Champions'
            WHEN rfm_score >= 0.5 THEN 'Loyal Customers'
            WHEN rfm_score >= 0.0 THEN 'Potential Loyalists'
            WHEN rfm_score >= -0.5 THEN 'Needs Attention'
            ELSE 'At Risk / Lost'
       END AS customer_segment
FROM rfm_final
ORDER BY rfm_score DESC;
