# Maven Fuzzy Factory - Dashboard Chart Prompts for BI Tools

## Chart 1: Revenue by Product
**BI Tool Prompt:**
Create a vertical bar chart showing total revenue by product name. Use the orders table joined with products table, aggregating price_usd by product_name. Order by revenue descending to highlight top performers. Include these metrics: total revenue, product name. Sort by revenue in descending order.

**Data Query:**
```
SELECT p.product_name, SUM(o.price_usd) as total_revenue
FROM orders o
JOIN products p ON o.primary_product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_revenue DESC
```

## Chart 2: Weekly Conversion Rate Trend
**BI Tool Prompt:**
Create a line chart showing conversion rate by week. Calculate conversion rate as: (orders per week ÷ sessions per week). Use created_at fields from both orders and website_sessions tables, grouping by week. Show date range on x-axis and conversion rate percentage on y-axis.

**Data Query:**
```
WITH weekly_orders AS (
  SELECT DATE_TRUNC('week', created_at) as week, COUNT(*) as orders
  FROM orders
  GROUP BY DATE_TRUNC('week', created_at)
),
weekly_sessions AS (
  SELECT DATE_TRUNC('week', created_at) as week, COUNT(*) as sessions
  FROM website_sessions
  GROUP BY DATE_TRUNC('week', created_at)
)
SELECT 
  wo.week,
  (wo.orders::DECIMAL / ws.sessions) as conversion_rate
FROM weekly_orders wo
JOIN weekly_sessions ws ON wo.week = ws.week
ORDER BY wo.week
```

## Chart 3: Revenue per Session by Channel
**BI Tool Prompt:**
Create a horizontal bar chart showing revenue per session for each marketing channel. Group website_sessions by utm_source, utm_campaign, and device_type. Calculate total revenue from linked orders divided by total sessions for each channel group. Display channel combinations on y-axis, RPS on x-axis.

**Data Query:**
```
SELECT 
  CONCAT(utm_source, ' - ', utm_campaign, ' - ', device_type) as channel,
  SUM(COALESCE(o.price_usd, 0)) / COUNT(ws.website_session_id) as rps
FROM website_sessions ws
LEFT JOIN orders o ON ws.website_session_id = o.website_session_id
GROUP BY utm_source, utm_campaign, device_type
ORDER BY rps DESC
```

## Chart 4: RPS by Session Type (New vs Repeat)
**BI Tool Prompt:**
Create a clustered bar chart comparing revenue per session for new vs repeat customers. Use the is_repeat_session field from website_sessions joined with orders table. Calculate total revenue divided by total sessions for each customer type category. X-axis: customer type, Y-axis: revenue per session.

**Data Query:**
```
SELECT 
  CASE WHEN is_repeat_session = 0 THEN 'New Customer' 
       ELSE 'Repeat Customer' END as customer_type,
  SUM(COALESCE(o.price_usd, 0)) / COUNT(ws.website_session_id) as rps
FROM website_sessions ws
LEFT JOIN orders o ON ws.website_session_id = o.website_session_id
GROUP BY is_repeat_session
ORDER BY is_repeat_session
```

## Chart 5: Normalized Product Performance
**BI Tool Prompt:**
Create a grouped bar chart showing normalized average price and order volume by product. Join orders with products table and calculate: 1) average price_usd by product_name, and 2) count of orders by product_name. Normalize both metrics to 0-1 scale for comparison. Two bars per product: one for average price, one for volume.

**Data Query:**
```
SELECT 
  p.product_name,
  AVG(o.price_usd) as avg_price,
  COUNT(o.order_id) as order_volume
FROM orders o
JOIN products p ON o.primary_product_id = p.product_id
GROUP BY p.product_name
ORDER BY avg_price DESC
```

## Chart 6: Revenue vs Refunds (Pie Chart)
**BI Tool Prompt:**
Create a pie chart showing the proportion of net revenue vs total refunds. Calculate total revenue from orders table and total refunds from order_item_refunds table. Show two segments: Total Revenue - Total Refunds (net revenue), and Total Refunds. Include percentage values on each segment.

**Data Query:**
```
SELECT 
  'Net Revenue' as category,
  (SELECT SUM(price_usd) FROM orders) - (SELECT SUM(refund_amount_usd) FROM order_item_refunds) as amount
UNION ALL
SELECT 
  'Refunds' as category,
  (SELECT SUM(refund_amount_usd) FROM order_item_refunds) as amount
```

## Additional Dashboard Formatting Instructions

### Color Scheme:
- Use a consistent, professional color palette (e.g., blue gradient for revenue, red for refunds)
- Ensure high contrast for accessibility
- Use the same colors across related charts for consistency

### Titles:
- Revenue by Product: "Revenue Distribution by Product Line"
- Weekly Conversion Rate Trend: "Weekly Conversion Rate Performance"
- Revenue per Session by Channel: "Channel Performance Analysis"
- RPS by Session Type: "Customer Type Revenue Comparison"
- Normalized Product Performance: "Product Price vs Volume Analysis"
- Revenue vs Refunds: "Revenue Protection: Net vs Refunded"

### Filters (if applicable in BI tool):
- Date range filter: Enable selection of date ranges for time-based charts
- Product filter: For product-specific analysis
- Channel filter: For marketing channel analysis