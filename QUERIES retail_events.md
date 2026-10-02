Q1. Basic Filtering – High-Value Products
Find all event records where the base_price is greater than 1,000.

Display:
- event_id
- store_id
- product_code
- base_price
- promo_type

Concepts:
SELECT, WHERE, comparison operators.

QUERY

use retail_events_db;

SELECT
    event_id,
    store_id,
    product_code,
    base_price,
    promo_type
FROM fact_events
WHERE base_price > 1000;
------------------------------------------------------------
Q2. Sorting Promotional Events
Display all events where quantity sold after the promotion was greater than 100.

Display:
- event_id
- product_code
- promo_type
- quantity_sold(before_promo)
- quantity_sold(after_promo)

Sort by quantity_sold(after_promo) in descending order.

Concepts:
WHERE, ORDER BY, DESC.

QUERY
SELECT
    event_id,
    product_code,
    promo_type,
    `quantity_sold(before_promo)`,
    `quantity_sold(after_promo)`
FROM fact_events
WHERE `quantity_sold(after_promo)` > 100
ORDER BY `quantity_sold(after_promo)` DESC;

------------------------------------------------------------
Q3. DISTINCT Promotion Types
Find all unique promotion types used in the dataset.

Display only the unique promo_type values.

Concepts:
DISTINCT.

QUERY
SELECT DISTINCT
    promo_type
FROM fact_events;

------------------------------------------------------------
Q4. Basic Aggregation
Calculate the following for the complete fact_events table:

- Total number of events
- Total quantity sold before promotion
- Total quantity sold after promotion
- Average base price
- Maximum base price
- Minimum base price

Return all metrics in one row.

Concepts:
COUNT, SUM, AVG, MAX, MIN.

QUERY

SELECT
    COUNT(event_id) AS total_events,
    SUM(`quantity_sold(before_promo)`) AS total_before,
    SUM(`quantity_sold(after_promo)`) AS total_after,
    AVG(base_price) AS average_base_price,
    MAX(base_price) AS maximum_base_price,
    MIN(base_price) AS minimum_base_price
FROM fact_events;

============================================================
MEDIUM QUESTIONS
============================================================

Q5. Sales Volume by Promotion Type
For each promo_type, calculate:

- Number of events
- Total quantity sold before promotion
- Total quantity sold after promotion

Sort by total quantity sold after promotion in descending order.

Concepts:
GROUP BY, COUNT, SUM, ORDER BY.

QUERY
SELECT
    promo_type,
    COUNT(event_id) AS event_count,
    SUM(`quantity_sold(before_promo)`) AS total_before,
    SUM(`quantity_sold(after_promo)`) AS total_after
FROM fact_events
GROUP BY promo_type
ORDER BY total_after DESC;

------------------------------------------------------------

Q6. Promotion Uplift
For every promotion type, calculate:

- Total quantity before promotion
- Total quantity after promotion
- Quantity increase/decrease

Use:

Quantity Change = After Promo Quantity - Before Promo Quantity

Display:
- promo_type
- total_before
- total_after
- quantity_change

Sort by quantity_change descending.

Concepts:
GROUP BY, SUM, arithmetic calculations, aliases.

QUERY
SELECT
    promo_type,
    SUM(`quantity_sold(before_promo)`) AS total_before,
    SUM(`quantity_sold(after_promo)`) AS total_after,
    SUM(`quantity_sold(after_promo)`)
        - SUM(`quantity_sold(before_promo)`) AS quantity_change
FROM fact_events
GROUP BY promo_type
ORDER BY quantity_change DESC;

------------------------------------------------------------

Q7. Product Performance
Using fact_events and dim_products, calculate total quantity sold after promotion for every product.

Display:
- product_code
- product_name
- category
- total quantity after promotion

Sort by total quantity after promotion descending.

Concepts:
INNER JOIN, GROUP BY, SUM, ORDER BY.

QUERY
SELECT
    fact_events.product_code,
    dim_products.product_name,
    dim_products.category,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_quantity_after
FROM fact_events
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY
    fact_events.product_code,
    dim_products.product_name,
    dim_products.category
ORDER BY total_quantity_after DESC;


------------------------------------------------------------

Q8. Category-Level Performance
Using fact_events and dim_products, calculate for every product category:

- Number of events
- Total quantity before promotion
- Total quantity after promotion
- Quantity change

Sort categories by total quantity after promotion descending.

Concepts:
JOIN, GROUP BY, SUM, COUNT, arithmetic calculations.

QUERY
SELECT
    dim_products.category,
    COUNT(fact_events.event_id) AS event_count,
    SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,
    SUM(fact_events.`quantity_sold(after_promo)`)
        - SUM(fact_events.`quantity_sold(before_promo)`) AS quantity_change
FROM fact_events
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY dim_products.category
ORDER BY total_after DESC;

------------------------------------------------------------

Q9. Store Performance
Using fact_events and dim_stores, calculate for every city:

- Number of promotional events
- Total quantity before promotion
- Total quantity after promotion

Display:
- city
- event_count
- total_before
- total_after

Sort cities by total_after descending.

Concepts:
JOIN, GROUP BY, aggregation, ORDER BY.

QUERY
SELECT
    dim_stores.city,
    COUNT(fact_events.event_id) AS event_count,
    SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after
FROM fact_events
INNER JOIN dim_stores
    ON fact_events.store_id = dim_stores.store_id
GROUP BY dim_stores.city
ORDER BY total_after DESC;

------------------------------------------------------------

Q10. Campaign Performance
Using fact_events and dim_campaigns, calculate for each campaign:

- Campaign name
- Start date
- End date
- Number of events
- Total quantity before promotion
- Total quantity after promotion

Sort by total quantity after promotion descending.

Concepts:
JOIN, GROUP BY, date columns, aggregation.

QUERY
SELECT
    dim_campaigns.campaign_name,
    dim_campaigns.start_date,
    dim_campaigns.end_date,
    COUNT(fact_events.event_id) AS event_count,
    SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after
FROM fact_events
INNER JOIN dim_campaigns
    ON fact_events.campaign_id = dim_campaigns.campaign_id
GROUP BY
    dim_campaigns.campaign_id,
    dim_campaigns.campaign_name,
    dim_campaigns.start_date,
    dim_campaigns.end_date
ORDER BY total_after DESC;

------------------------------------------------------------

Q11. Product Category with HAVING
Find product categories where the total quantity sold after promotion is greater than 1,000.

Display:
- category
- total quantity after promotion
- average base price

Sort by total quantity after promotion descending.

Concepts:
JOIN, GROUP BY, HAVING, AVG, SUM.

QUERY

SELECT
    dim_products.category,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,
    AVG(fact_events.base_price) AS average_base_price
FROM fact_events
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY dim_products.category
HAVING SUM(fact_events.`quantity_sold(after_promo)`) > 1000
ORDER BY total_after DESC;

------------------------------------------------------------

Q12. Store + Category Analysis
Using fact_events, dim_stores and dim_products, calculate total quantity sold after promotion for every combination of:

- City
- Product category

Display:
- city
- category
- total quantity after promotion

Sort first by city and then by total quantity descending.

Concepts:
Multiple JOINs, GROUP BY, ORDER BY.

QUERY

SELECT
    dim_stores.city,
    dim_products.category,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_quantity_after
FROM fact_events
INNER JOIN dim_stores
    ON fact_events.store_id = dim_stores.store_id
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY
    dim_stores.city,
    dim_products.category
ORDER BY
    dim_stores.city,
    total_quantity_after DESC;

------------------------------------------------------------

Q13. Promotion Effectiveness by Product
For each product, calculate:

- Product name
- Category
- Total quantity before promotion
- Total quantity after promotion
- Quantity change
- Percentage change

Use:

Percentage Change =
((After Promo - Before Promo) / Before Promo) * 100

Handle division by zero appropriately.

Sort by percentage change descending.

Concepts:
JOIN, GROUP BY, arithmetic calculations, NULLIF, percentage calculations.

QUERY

SELECT
    dim_products.product_name,
    dim_products.category,
    SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,
    SUM(fact_events.`quantity_sold(after_promo)`)
        - SUM(fact_events.`quantity_sold(before_promo)`) AS quantity_change,
    (
        (
            SUM(fact_events.`quantity_sold(after_promo)`)
            - SUM(fact_events.`quantity_sold(before_promo)`)
        )
        /
        NULLIF(
            SUM(fact_events.`quantity_sold(before_promo)`),
            0
        )
    ) * 100 AS percentage_change
FROM fact_events
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY
    fact_events.product_code,
    dim_products.product_name,
    dim_products.category
ORDER BY percentage_change DESC;

------------------------------------------------------------

Q14. Campaign and Promotion Type Analysis
For each campaign and promo_type combination, calculate:

- Number of events
- Total quantity before promotion
- Total quantity after promotion
- Quantity change

Display:
- campaign_name
- promo_type
- event_count
- total_before
- total_after
- quantity_change

Sort by campaign_name and quantity_change descending.

Concepts:
Multiple GROUP BY columns, JOIN, aggregation, ORDER BY.

QUERY

SELECT
    dim_campaigns.campaign_name,
    fact_events.promo_type,
    COUNT(fact_events.event_id) AS event_count,
    SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
    SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,
    SUM(fact_events.`quantity_sold(after_promo)`)
        - SUM(fact_events.`quantity_sold(before_promo)`) AS quantity_change
FROM fact_events
INNER JOIN dim_campaigns
    ON fact_events.campaign_id = dim_campaigns.campaign_id
GROUP BY
    dim_campaigns.campaign_name,
    fact_events.promo_type
ORDER BY
    dim_campaigns.campaign_name,
    quantity_change DESC;

------------------------------------------------------------

Q15. Product Revenue Before and After Promotion
For each product, calculate:

1. Revenue before promotion =
   base_price × quantity_sold(before_promo)

2. Revenue after promotion =
   base_price × quantity_sold(after_promo)

3. Revenue difference =
   Revenue after - Revenue before

Display:
- product_name
- category
- revenue_before
- revenue_after
- revenue_difference

Sort by revenue_difference descending.

Concepts:
JOIN, GROUP BY, SUM, arithmetic calculations, aliases.

QUERY

SELECT
    dim_products.product_name,
    dim_products.category,
    SUM(
        fact_events.base_price *
        fact_events.`quantity_sold(before_promo)`
    ) AS revenue_before,
    SUM(
        fact_events.base_price *
        fact_events.`quantity_sold(after_promo)`
    ) AS revenue_after,
    SUM(
        fact_events.base_price *
        fact_events.`quantity_sold(after_promo)`
    )
    -
    SUM(
        fact_events.base_price *
        fact_events.`quantity_sold(before_promo)`
    ) AS revenue_difference
FROM fact_events
INNER JOIN dim_products
    ON fact_events.product_code = dim_products.product_code
GROUP BY
    fact_events.product_code,
    dim_products.product_name,
    dim_products.category
ORDER BY revenue_difference DESC;

------------------------------------------------------------

Q16. Classify Promotion Performance
For every promotion type, calculate total quantity before and after promotion.

Then classify the promotion using CASE:

- Percentage change >= 50% → "High Impact"
- Percentage change >= 20% → "Medium Impact"
- Percentage change < 20% → "Low Impact"

Display:
- promo_type
- total_before
- total_after
- percentage_change
- performance_category

Sort by percentage_change descending.

Concepts:
GROUP BY, CASE, arithmetic calculations, NULLIF, aliases.

QUERY

SELECT
    promo_type,
    SUM(`quantity_sold(before_promo)`) AS total_before,
    SUM(`quantity_sold(after_promo)`) AS total_after,
    (
        (
            SUM(`quantity_sold(after_promo)`)
            - SUM(`quantity_sold(before_promo)`)
        )
        /
        NULLIF(
            SUM(`quantity_sold(before_promo)`),
            0
        )
    ) * 100 AS percentage_change,
    CASE
        WHEN (
            (
                SUM(`quantity_sold(after_promo)`)
                - SUM(`quantity_sold(before_promo)`)
            )
            /
            NULLIF(
                SUM(`quantity_sold(before_promo)`),
                0
            )
        ) * 100 >= 50
            THEN 'High Impact'

        WHEN (
            (
                SUM(`quantity_sold(after_promo)`)
                - SUM(`quantity_sold(before_promo)`)
            )
            /
            NULLIF(
                SUM(`quantity_sold(before_promo)`),
                0
            )
        ) * 100 >= 20
            THEN 'Medium Impact'

        ELSE 'Low Impact'
    END AS performance_category
FROM fact_events
GROUP BY promo_type
ORDER BY percentage_change DESC;

============================================================
HARD QUESTIONS
============================================================

Q17. Top Products Within Each Category
Using a CTE:

1. Calculate total quantity sold after promotion for every product.
2. Rank products within each category based on total quantity sold after promotion.
3. Return only the top 2 products from every category.

Display:
- category
- product_name
- total_quantity_after
- category_rank

Concepts:
CTE, JOIN, GROUP BY, DENSE_RANK/ROW_NUMBER,
PARTITION BY, window functions.

QUERY

WITH product_totals AS (
    SELECT
        dim_products.category,
        dim_products.product_name,
        SUM(fact_events.`quantity_sold(after_promo)`) AS total_quantity_after
    FROM fact_events
    INNER JOIN dim_products
        ON fact_events.product_code = dim_products.product_code
    GROUP BY
        fact_events.product_code,
        dim_products.product_name,
        dim_products.category
),
ranked_products AS (
    SELECT
        category,
        product_name,
        total_quantity_after,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY total_quantity_after DESC
        ) AS category_rank
    FROM product_totals
)
SELECT
    category,
    product_name,
    total_quantity_after,
    category_rank
FROM ranked_products
WHERE category_rank <= 2;
------------------------------------------------------------

Q18. Best-Performing Stores Within Each City
Calculate total quantity sold after promotion for each store.

Join dim_stores to obtain the city.

Then rank stores within each city based on total quantity sold after promotion.

Return the top 2 stores from each city.

Display:
- city
- store_id
- total_quantity_after
- city_rank

Concepts:
JOIN, CTE, GROUP BY, window functions,
PARTITION BY, RANK/DENSE_RANK.

QUERY

WITH store_totals AS (
    SELECT
        dim_stores.city,
        fact_events.store_id,
        SUM(fact_events.`quantity_sold(after_promo)`) AS total_quantity_after
    FROM fact_events
    INNER JOIN dim_stores
        ON fact_events.store_id = dim_stores.store_id
    GROUP BY
        dim_stores.city,
        fact_events.store_id
),
ranked_stores AS (
    SELECT
        city,
        store_id,
        total_quantity_after,
        ROW_NUMBER() OVER (
            PARTITION BY city
            ORDER BY total_quantity_after DESC
        ) AS city_rank
    FROM store_totals
)
SELECT
    city,
    store_id,
    total_quantity_after,
    city_rank
FROM ranked_stores
WHERE city_rank <= 2;
------------------------------------------------------------

Q19. Campaign-Level Product Performance
For every campaign and product:

Calculate:
- Total quantity before promotion
- Total quantity after promotion
- Quantity change
- Percentage change

Then rank products within each campaign based on percentage change.

Return the top 3 products for every campaign.

Display:
- campaign_name
- product_name
- total_before
- total_after
- quantity_change
- percentage_change
- campaign_rank

Concepts:
Multiple JOINs, CTE, GROUP BY, arithmetic calculations,
NULLIF, window functions, PARTITION BY, ranking.

QUERY

WITH campaign_product_totals AS (
    SELECT
        dim_campaigns.campaign_name,
        dim_products.product_name,
        SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,
        SUM(fact_events.`quantity_sold(after_promo)`) AS total_after
    FROM fact_events
    INNER JOIN dim_campaigns
        ON fact_events.campaign_id = dim_campaigns.campaign_id
    INNER JOIN dim_products
        ON fact_events.product_code = dim_products.product_code
    GROUP BY
        fact_events.campaign_id,
        fact_events.product_code,
        dim_campaigns.campaign_name,
        dim_products.product_name
),
campaign_product_metrics AS (
    SELECT
        campaign_name,
        product_name,
        total_before,
        total_after,
        total_after - total_before AS quantity_change,
        (
            (total_after - total_before)
            / NULLIF(total_before, 0)
        ) * 100 AS percentage_change
    FROM campaign_product_totals
),
ranked_products AS (
    SELECT
        campaign_name,
        product_name,
        total_before,
        total_after,
        quantity_change,
        percentage_change,
        ROW_NUMBER() OVER (
            PARTITION BY campaign_name
            ORDER BY percentage_change DESC
        ) AS campaign_rank
    FROM campaign_product_metrics
)
SELECT
    campaign_name,
    product_name,
    total_before,
    total_after,
    quantity_change,
    percentage_change,
    campaign_rank
FROM ranked_products
WHERE campaign_rank <= 3;

WITH product_metrics AS (
    SELECT
        fact_events.product_code,
        dim_products.product_name,
        dim_products.category,

        COUNT(fact_events.event_id) AS event_count,

        SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,

        SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,

        SUM(
            fact_events.base_price *
            fact_events.`quantity_sold(before_promo)`
        ) AS revenue_before,

        SUM(
            fact_events.base_price *
            fact_events.`quantity_sold(after_promo)`
        ) AS revenue_after,

        AVG(fact_events.base_price) AS average_base_price

    FROM fact_events

    INNER JOIN dim_products
        ON fact_events.product_code = dim_products.product_code

    GROUP BY
        fact_events.product_code,
        dim_products.product_name,
        dim_products.category
),

product_calculations AS (
    SELECT
        product_code,
        product_name,
        category,
        event_count,
        total_before,
        total_after,

        total_after - total_before AS quantity_change,

        CASE
            WHEN total_before = 0 THEN NULL
            ELSE
                (
                    (total_after - total_before)
                    / NULLIF(total_before, 0)
                ) * 100
        END AS percentage_change,

        revenue_before,
        revenue_after,

        revenue_after - revenue_before AS revenue_change,

        average_base_price

    FROM product_metrics
),

ranked_products AS (
    SELECT
        product_name,
        category,
        event_count,
        total_before,
        total_after,
        quantity_change,
        percentage_change,
        revenue_before,
        revenue_after,
        revenue_change,
        average_base_price,

        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue_change DESC
        ) AS product_rank

    FROM product_calculations
)

SELECT
    product_name,
    category,
    event_count,
    total_before,
    total_after,
    quantity_change,
    percentage_change,
    revenue_before,
    revenue_after,
    revenue_change,
    average_base_price,
    product_rank
FROM ranked_products
WHERE product_rank <= 2;

------------------------------------------------------------

Q20. Complete Promotional Performance Analysis
Create a complete analytical report at the product-category level.

For every product, calculate:

- Product name
- Category
- Number of promotional events
- Total quantity before promotion
- Total quantity after promotion
- Quantity change
- Percentage change
- Revenue before promotion
- Revenue after promotion
- Revenue change
- Average base price
- Product rank within its category

Use:

Quantity Change =
Total After - Total Before

Percentage Change =
((Total After - Total Before) / Total Before) * 100

Revenue Before =
SUM(base_price × quantity_before)

Revenue After =
SUM(base_price × quantity_after)

Revenue Change =
Revenue After - Revenue Before

Then:

1. Rank products within each category by Revenue Change.
2. Return only the top 2 products from each category.
3. Use appropriate handling for division by zero.

QUERY

WITH product_metrics AS (
    SELECT
        fact_events.product_code,
        dim_products.product_name,
        dim_products.category,

        COUNT(fact_events.event_id) AS event_count,

        SUM(fact_events.`quantity_sold(before_promo)`) AS total_before,

        SUM(fact_events.`quantity_sold(after_promo)`) AS total_after,

        SUM(
            fact_events.base_price *
            fact_events.`quantity_sold(before_promo)`
        ) AS revenue_before,

        SUM(
            fact_events.base_price *
            fact_events.`quantity_sold(after_promo)`
        ) AS revenue_after,

        AVG(fact_events.base_price) AS average_base_price

    FROM fact_events

    INNER JOIN dim_products
        ON fact_events.product_code = dim_products.product_code

    GROUP BY
        fact_events.product_code,
        dim_products.product_name,
        dim_products.category
),

product_calculations AS (
    SELECT
        product_code,
        product_name,
        category,
        event_count,
        total_before,
        total_after,

        total_after - total_before AS quantity_change,

        CASE
            WHEN total_before = 0 THEN NULL
            ELSE
                (
                    (total_after - total_before)
                    / NULLIF(total_before, 0)
                ) * 100
        END AS percentage_change,

        revenue_before,
        revenue_after,

        revenue_after - revenue_before AS revenue_change,

        average_base_price

    FROM product_metrics
),

ranked_products AS (
    SELECT
        product_name,
        category,
        event_count,
        total_before,
        total_after,
        quantity_change,
        percentage_change,
        revenue_before,
        revenue_after,
        revenue_change,
        average_base_price,

        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue_change DESC
        ) AS product_rank

    FROM product_calculations
)

SELECT
    product_name,
    category,
    event_count,
    total_before,
    total_after,
    quantity_change,
    percentage_change,
    revenue_before,
    revenue_after,
    revenue_change,
    average_base_price,
    product_rank
FROM ranked_products
WHERE product_rank <= 2;

------------------------------------------------------------
------------------------------------------------------------



