---
title: E-commerce Dashboard
---

```sql date_range
  SELECT 
    DISTINCT(order_purchase_timestamp) AS date
  FROM 
    ecommerce_project.orders
```


```sql total_rev
SELECT
    SUM(payment_value) as total_revenue
FROM 
    ecommerce_project.payments p
JOIN 
    ecommerce_project.orders o
  ON p.order_id = o.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
```

```sql total_orders
SELECT
    COUNT(DISTINCT(order_id)) as total_orders,
    COUNT(DISTINCT(customer_id)) as total_customers
FROM
    ecommerce_project.orders
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
```

```sql avg_order_value
SELECT
    SUM(payment_value)/COUNT(DISTINCT(p.order_id)) as avg_order_value,
FROM
    ecommerce_project.payments p
JOIN 
    ecommerce_project.orders o
  ON p.order_id = o.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
```

```sql delivery_time
SELECT 
    ROUND(AVG(delivery_time_days), 1) AS avg_delivery_days
FROM 
    ecommerce_project.orders
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
```


```sql repeat_customers
SELECT COUNT(*) AS repeat_customers
FROM (
    SELECT customer_id
    FROM ecommerce_project.orders
    WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
     GROUP BY customer_id
    HAVING COUNT(order_id) > 1
)
```

```sql daily_rev
SELECT
  CONCAT(
  CAST(EXTRACT(YEAR FROM o.order_purchase_timestamp) AS STRING),
  '-',
  LPAD(CAST(EXTRACT(MONTH FROM o.order_purchase_timestamp) AS STRING), 2, '0'),
  '-',
  LPAD(CAST(EXTRACT(DAY FROM o.order_purchase_timestamp) AS STRING), 2, '0')
) AS day,
  ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.orders o
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY day
ORDER BY day
```

```sql daily_orders
SELECT
    CONCAT(
      CAST(EXTRACT(YEAR FROM order_purchase_timestamp) AS STRING), '-',
      LPAD(CAST(EXTRACT(MONTH FROM order_purchase_timestamp) AS STRING), 2, '0'), '-',
      LPAD(CAST(EXTRACT(DAY FROM order_purchase_timestamp) AS STRING), 2, '0')
    ) AS day,
    COUNT(order_id) AS order_count
FROM ecommerce_project.orders
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY day
ORDER BY day
```

```sql aov
SELECT
  CONCAT(
    CAST(EXTRACT(YEAR FROM o.order_purchase_timestamp) AS STRING), '-',
    LPAD(CAST(EXTRACT(MONTH FROM o.order_purchase_timestamp) AS STRING), 2, '0'), '-',
    LPAD(CAST(EXTRACT(DAY FROM o.order_purchase_timestamp) AS STRING), 2, '0')
  ) AS day,
  ROUND(SUM(p.payment_value) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM ecommerce_project.orders o
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY day
ORDER BY day
```

```sql product_category
SELECT 
  REPLACE(UPPER(SUBSTR(pr.product_category_name, 1, 1)) || 
        LOWER(SUBSTR(pr.product_category_name, 2)), '_', ' ') AS category,
  COUNT(oi.product_id) AS total_products_sold
FROM ecommerce_project.order_items oi
JOIN ecommerce_project.products pr ON oi.product_id = pr.product_id
JOIN ecommerce_project.orders o ON oi.order_id = o.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY category
ORDER BY total_products_sold DESC
LIMIT 10
```

```sql product_revenue
WITH order_item_revenue AS (
  SELECT
    oi.order_id,
    pr.product_category_name AS category,
    oi.price,
    SUM(oi.price) OVER (PARTITION BY oi.order_id) AS order_total_price
  FROM ecommerce_project.order_items oi
  JOIN ecommerce_project.products pr ON oi.product_id = pr.product_id
),

payment_per_order AS (
  SELECT
    order_id,
    SUM(payment_value) AS total_payment
  FROM ecommerce_project.payments
  GROUP BY order_id
)

SELECT 
  REPLACE(
    UPPER(SUBSTR(category, 1, 1)) || LOWER(SUBSTR(category, 2)),
    '_', ' '
  ) AS category,
  ROUND(SUM((oi.price / oi.order_total_price) * p.total_payment), 2) AS total_revenue
FROM order_item_revenue oi
JOIN payment_per_order p ON oi.order_id = p.order_id
WHERE oi.order_id IN (
  SELECT order_id FROM ecommerce_project.orders
  WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
)
GROUP BY category
ORDER BY total_revenue DESC
LIMIT 10
```



```sql payment_type
SELECT 
    REPLACE(
    UPPER(SUBSTR(p.payment_type, 1, 1)) || 
    LOWER(SUBSTR(p.payment_type, 2)),'_', ' ') AS payment_type,
    COUNT(*) AS usage_count,
    ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.payments p
JOIN ecommerce_project.orders o ON p.order_id = o.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY p.payment_type
ORDER BY usage_count DESC
```

```sql state_rev
SELECT 
  c.customer_state AS state,
  ROUND(SUM(p.payment_value), 2) AS revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY state
ORDER BY revenue DESC
LIMIT 10
```


```sql city_rev
SELECT 
  UPPER(SUBSTR(c.customer_city, 1, 1)) || LOWER(SUBSTR(c.customer_city, 2)) AS city,
  ROUND(SUM(p.payment_value), 2) AS revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY city
ORDER BY revenue DESC
LIMIT 10
```

```sql customers_by_state
SELECT 
  customer_state AS state,
  COUNT(DISTINCT c.customer_id) AS total_customers,
  COUNT(DISTINCT o.order_id) AS total_orders,
  ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY state
ORDER BY total_revenue DESC
LIMIT 10
```

```sql customers_by_city
SELECT 
  UPPER(SUBSTR(c.customer_city, 1, 1)) || LOWER(SUBSTR(c.customer_city, 2)) AS city,
  c.customer_state AS state,
  COUNT(DISTINCT c.customer_id) AS total_customers,
  COUNT(DISTINCT o.order_id) AS total_orders,
  ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY city, state
ORDER BY total_revenue DESC
LIMIT 10
```

```sql state_summary
SELECT 
  customer_state AS state,
  COUNT(DISTINCT c.customer_id) AS total_customers,
  COUNT(DISTINCT o.order_id) AS total_orders,
  ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY state
ORDER BY total_revenue DESC
```

```sql city_summary
SELECT 
  UPPER(SUBSTR(c.customer_city, 1, 1)) || LOWER(SUBSTR(c.customer_city, 2)) AS city,
  c.customer_state AS state,
  COUNT(DISTINCT c.customer_id) AS total_customers,
  COUNT(DISTINCT o.order_id) AS total_orders,
  ROUND(SUM(p.payment_value), 2) AS total_revenue
FROM ecommerce_project.customers c
JOIN ecommerce_project.orders o ON c.customer_id = o.customer_id
JOIN ecommerce_project.payments p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY city, state
ORDER BY total_revenue DESC
```


```sql top_sellers_revenue
SELECT 
  seller_id,
  ROUND(SUM(oi.price), 2) AS total_revenue
FROM ecommerce_project.order_items oi
JOIN ecommerce_project.orders o ON oi.order_id = o.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY seller_id
ORDER BY total_revenue DESC
LIMIT 10
```

```sql top_sellers_items
SELECT 
  seller_id,
  COUNT(*) AS total_items_sold
FROM ecommerce_project.order_items oi
JOIN ecommerce_project.orders o ON oi.order_id = o.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY seller_id
ORDER BY total_items_sold DESC
LIMIT 10
```

```sql top_sellers_table
SELECT 
  seller_id,
  ROUND(SUM(oi.price), 2) AS total_revenue,
  COUNT(*) AS total_items_sold
FROM ecommerce_project.order_items oi
JOIN ecommerce_project.orders o ON oi.order_id = o.order_id
WHERE o.order_purchase_timestamp 
  BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
GROUP BY seller_id
ORDER BY total_revenue DESC
```

```sql delivery_time
SELECT 
  CONCAT(
    CAST(EXTRACT(YEAR FROM order_purchase_timestamp) AS STRING), '-',
    LPAD(CAST(EXTRACT(MONTH FROM order_purchase_timestamp) AS STRING), 2, '0'), '-',
    LPAD(CAST(EXTRACT(DAY FROM order_purchase_timestamp) AS STRING), 2, '0')) AS day,
  ROUND(AVG(delivery_time_days), 2) AS avg_delivery_days
FROM ecommerce_project.orders
WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
  AND delivery_time_days IS NOT NULL
GROUP BY day
ORDER BY day
```

```sql product_rev_table
WITH order_item_revenue AS (
  SELECT
    oi.order_id,
    pr.product_category_name AS category,
    oi.price,
    SUM(oi.price) OVER (PARTITION BY oi.order_id) AS order_total_price
  FROM ecommerce_project.order_items oi
  JOIN ecommerce_project.products pr ON oi.product_id = pr.product_id
),

payment_per_order AS (
  SELECT
    order_id,
    SUM(payment_value) AS total_payment
  FROM ecommerce_project.payments
  GROUP BY order_id
)

SELECT 
  REPLACE(
    UPPER(SUBSTR(category, 1, 1)) || LOWER(SUBSTR(category, 2)),
    '_', ' '
  ) AS category,
  COUNT(*) AS total_units_sold,
  ROUND(SUM((oi.price / oi.order_total_price) * p.total_payment), 2) AS total_revenue,
  ROUND(SUM((oi.price / oi.order_total_price) * p.total_payment) / COUNT(*), 2) AS avg_revenue_per_unit
FROM order_item_revenue oi
JOIN payment_per_order p ON oi.order_id = p.order_id
WHERE oi.order_id IN (
  SELECT order_id FROM ecommerce_project.orders
  WHERE order_purchase_timestamp BETWEEN '${inputs.date_filter.start}' AND '${inputs.date_filter.end}'
)
GROUP BY category
ORDER BY total_revenue DESC
```




<DateRange 
name="date_filter"
data={date_range} 
dates=date/>



##  KPI Overview

<Grid cols=3>
<BigValue
    data={total_rev}
    value=total_revenue
    fmt=usd0
    title="Total Revenue"
/>

<BigValue
    data={total_orders}
    value=total_orders
    fmt=num0
    title="Total Orders"
/>

<BigValue
    data={total_orders}
    value=total_customers
    fmt=num0
    title="Total Customers"
/>
</Grid>

<Grid cols=3>
<BigValue
    data={avg_order_value}
    value=avg_order_value
    fmt=usd2
    title="Average Order Value"
/>

<BigValue
    data={delivery_time}
    value=avg_delivery_days
    fmt=num0
    title="Average Days for Delivery"
/>


<BigValue
    data={repeat_customers}
    value=repeat_customers
    fmt=num0
    title="Repeat Customers"
/>
</Grid>

<Alert status="info">
💡 Repeat customer count is 0. This result is expected due to the absence of historical purchase behavior in the dataset.
</Alert>


## Sales & Performance Trends

<Grid cols=2>
<LineChart
    data={daily_rev}
    x=day
    y=total_revenue
    sort=false
    title="Revenue Over Time"
/>

<LineChart
    data={daily_orders}
    x=day
    y=order_count
    sort=false
    title="Orders Over Time"
/>

</Grid>

<LineChart
    data={aov}
    x=day
    y=avg_order_value
    sort=false
    title="Average Order Value"
/>

<LineChart
  data={delivery_time}
  x=day
  y=avg_delivery_days
  sort=false
  title="Average Delivery Time Over Time"
/>

<Alert status="info">
💡 Average delivery time is decreasing, but so are revenue and order volumes. This may be due to lighter fulfillment loads rather than operational improvements.
</Alert>



## Seller Performance

<Grid cols=2>
<BarChart 
    data={top_sellers_revenue}
    x="seller_id"
    y="total_revenue"
   title="Top Sellers by Revenue"
    yFmt=usd0
/>

<BarChart 
  data={top_sellers_items}
  x="seller_id"
  y="total_items_sold"
  title="Top Sellers by Items Sold"
  yFmt=num0
/>

</Grid>

<DataTable 
  data={top_sellers_table} 
  title="All Sellers: Revenue and Items Sold"
  search=true
/>

<Alert status="info">
💡 The top two sellers contribute nearly 1 in 5 dollars of total revenue. They are key drivers of performance and candidates for strategic partnerships or incentives.
</Alert>


## Product Performance

<BarChart
    data={product_category}
    x=category
    y=total_products_sold
    title="Top 10 Product Categories Sold"
    yFmt=num0
/>


<BarChart
  data={product_revenue}
  x=category
  y=total_revenue
  title="Top 10 Product Categories by Revenue"
  yFmt=usd2
/>


<DataTable 
  data={product_rev_table}
  title="Revenue Breakdown by Product Category"
  search=true
>
  <Column id=category />
  <Column id=total_units_sold fmt=num0 />
  <Column id=total_revenue fmt=usd2 />
  <Column id=avg_revenue_per_unit fmt=usd2 />
</DataTable>

<Alert status="info">
💡 Toys dominate both in volume (1.84M units) and revenue ($17.5M), far outpacing all other categories. Meanwhile, Housewares and Sports Leisure show high average revenue per unit (>$20), suggesting strong profitability despite lower volume.
</Alert>

## Payment Method Breakdown

<Grid cols = 2>
<BarChart 
  data={payment_type}
  x=payment_type
  y=usage_count
  title="Orders by Payment Method"
  yFmt=num0
  swapXY=true
/>

<BarChart 
  data={payment_type}
  x=payment_type
  y=total_revenue
  yFmt=usd2
  title="Revenue by Payment Method"
/>
</Grid>



<DataTable data={payment_type} search = true> 
	<Column id=payment_type/> 
	<Column id=usage_count fmt=num0/> 
	<Column id=total_revenue fmt=usd2/> 
</DataTable>

Information on Wallets and Vouchers → <Info description="Wallets are digital payment methods like app-based balances or stored credits (e.g., PayPal). Vouchers reflect discounts or promotional codes applied during checkout, often linked to marketing campaigns or incentives." color="info"/>

<Alert status="info">
💡 Wallets are used in ~18% of orders and generate over $4.5M in revenue, but adoption lags behind credit cards. Consider wallet-focused incentives, loyalty perks, or one-click reorders to increase usage.
</Alert>


## Customer Demographics & Location
<Grid cols=2>
  <BarChart 
    data={state_rev}
    x=state
    y=revenue
    title="Top 10 States by Revenue"
    yFmt=usd0
  />

 <BarChart 
    data={city_rev}
    x=city
    y=revenue
    title="Top 10 Cities by Revenue"
    yFmt=usd0
  />
</Grid>


<Grid cols=2>
<BarChart 
  data={customers_by_state}
  x="state"
  y="total_customers"
  title="Top 10 States by Customer Count"
  yFmt=num0
  swapXY=true
/>

<BarChart 
  data={customers_by_city}
  x="city"
  y="total_customers"
  title="Top 10 Cities by Customer Count"
  yFmt=num0
  swapXY=true
/>
</Grid>



<DataTable data={state_summary} search = true title="Customer & Revenue Breakdown by State">  
	<Column id=state/> 
	<Column id=total_customers/> 
	<Column id=total_orders/> 
	<Column id=total_revenue fmt=usd2/> 
</DataTable>


<DataTable data={city_summary} search = true title="Customer & Revenue Breakdown by City"> 
    <Column id=city/> 
	<Column id=state/> 
	<Column id=total_customers/> 
	<Column id=total_orders/> 
	<Column id=total_revenue fmt=usd2/> 
</DataTable>


<Alert status="info">
💡  São Paulo city alone drives nearly $3.7M in revenue, far ahead of other cities. Urban hubs like Rio de Janeiro and Belo Horizonte follow, reinforcing the importance of metropolitan targeting for marketing and logistics.
</Alert>



