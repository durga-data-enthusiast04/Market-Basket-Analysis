# Market-Basket-Analysis
A SQL-based project analyzing 1 year of Adobe Analytics data to identify high-performing product and category pairings using support, confidence, and lift.  💡 Used for cross-sell insights and boosting AOV in a fashion ecomm setup.
🔍 Just wrapped up a deep dive into Market Basket Analysis for a luxury fashion brand exploring one year of data with SQL + Adobe Analytics — and it has been one insightful ride!

✨ Why I started this?
To understand what products are often bought together and how that knowledge can drive better bundling, cross-sell opportunities, and campaign planning.

💡 Why SQL? Fast, clean, and directly can be connected to the raw data — perfect for pattern detection at scale without extra layers.

📌 Key Responsibilities (KRAs):
I extracted one year of data from Adobe Analytics Data Warehouse into Excel. You can use any data source, as long as it includes key parameters like Order ID, Product Category, Product Name, and Order details.
Wrote optimized SQL queries to extract meaningful product and category pairs
Built lift, confidence and support metrics to assess correlation strength
Translated data insights into merchandising recommendations

📊 Key Performance Indicators (KPIs) Monitored:
Pair order count/ Frequency (How often 2 products/categories are bought together)
Support (Proportion of transactions containing the pair)
Lift (Likelihood of co-purchase vs. random chance)
where Lift > 1: A and B are bought together more than by chance — strong positive relationship.
Lift = 1: No relationship — buying A does not affect B.
Lift < 1: Buying A makes buying B less likely — negative relationship.
Confidence (Shows how likely a second product is bought when a first one is)

📈Insights and Recommendations:
Created a prioritized list of product pairs for marketing bundles
Enabled targeted “Frequently Bought Together” campaigns on the website.
Helped inform product placement and bundling strategies to boost AOV.

🚀 My Impact:
💰 AOV increased by 16%
🛍️ Improved product pair visibility during purchase journey.

**For Product Pairs**
WITH total_orders AS (
    SELECT COUNT(DISTINCT `Order ID`) AS total_orders 
    FROM sales_data
),
product_counts AS (
    SELECT 
        `Product Name` AS product,
        COUNT(DISTINCT `Order ID`) AS order_count
    FROM sales_data
    GROUP BY `Product Name`
),
product_pairs AS (
    SELECT 
        a.`Product Name` AS product_A,
        b.`Product Name` AS product_B,
        COUNT(DISTINCT a.`Order ID`) AS pair_order_count
    FROM sales_data a
    JOIN sales_data b 
      ON a.`Order ID` = b.`Order ID`
    AND a.`Product Name` <= b.`Product Name`
    GROUP BY a.`Product Name`, b.`Product Name`
)

SELECT 
    pp.product_A,
    pp.product_B,
    pp.pair_order_count,
    ROUND(pp.pair_order_count / t.total_orders, 4) AS support,
	ROUND(pp.pair_order_count / pa.order_count, 4) AS confidence, -- Confidence of B given A
    ROUND((pp.pair_order_count / pa.order_count) / (pb.order_count / t.total_orders), 4) AS lift
FROM product_pairs pp
JOIN total_orders t
JOIN product_counts pa ON pa.product = pp.product_A
JOIN product_counts pb ON pb.product = pp.product_B
ORDER BY pp.pair_order_count DESC
LIMIT 1000;

**For Product Category Pairs**
WITH total_orders AS (
    SELECT COUNT(DISTINCT `Order ID`) AS total_orders 
    FROM sales_data
),
category_counts AS (
    SELECT 
        `Product Category` AS category,
        COUNT(DISTINCT `Order ID`) AS order_count
    FROM sales_data
    GROUP BY `Product Category`
),
category_pairs AS (
    SELECT 
        a.`Product Category` AS category_A,
        b.`Product Category` AS category_B,
        COUNT(DISTINCT a.`Order ID`) AS pair_order_count
    FROM sales_data a
    JOIN sales_data b 
      ON a.`Order ID` = b.`Order ID`
    AND a.`Product Category` <= b.`Product Category`
    GROUP BY a.`Product Category`, b.`Product Category`
)

SELECT 
    cp.category_A,
    cp.category_B,
    cp.pair_order_count,
    ROUND(cp.pair_order_count / t.total_orders, 4) AS support,
	ROUND(pp.pair_order_count / pa.order_count, 4) AS confidence, -- Confidence of B given A
    ROUND((cp.pair_order_count / ca.order_count) / (cb.order_count / t.total_orders), 4) AS lift
FROM category_pairs cp
JOIN total_orders t
JOIN category_counts ca ON ca.category = cp.category_A
JOIN category_counts cb ON cb.category = cp.category_B
ORDER BY cp.pair_order_count DESC
LIMIT 1000;
