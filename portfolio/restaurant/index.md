Restaurant Order Analysis
=========================
Kristen Healy  
2024-05-20


- [Assignment](#assignment)
- [Executive Summary](#executive-summary)
- [Detailed Analysis](#detailed-analysis)
  - [Menu Items](#menu-items-table)
  - [Orders](#orders-table)
  - [Combined Data](#combined-data)

## Assignment
The fictional Taste of the World Café, a restaurant serving international cuisine, debuted a new menu at the beginning of the year. The business owner wants to get an understanding of how the new menu items are doing with customers: which items are doing well and which are not, and what its top customers seem to like best.

Because the owner wants the information quickly and the data is in a MySQL database, all of the analysis will be done in SQL.

## Executive Summary
The menu has 32 items across 4 different international cuisines:
- Italian (9)
- Asian (8)
- Mexican (9)
- American (6)

Prices range from $5 (Edamame) to $19.95 (Shrimp Scampi), with an average menu item price of $13.29. Italian is the highest-priced category on average ($16.75), followed by Asian ($13.48). 

**Note:** I'm using "top orders by total" as a proxy for "top customers" since we have order- and item-level data but not customer-level data.

## Detailed Analysis
The restaurant consists of 2 tables:
- menu_items
- order_detail

### Menu Items Table

```
/* set restaurant_db as the my default db */
USE restaurant_db;

/* preliminary view of the table */
SELECT *
FROM menu_items;
```
![menu_items table](images/menu_items.png)

```
/* number of menu items and categories */
SELECT
    COUNT(menu_item_id) AS num_items,
    COUNT(DISTINCT category) AS num_categories
FROM menu_items;
```
![number of items and categories](images/ItemCatCount.png)
```
/* Summary menu item pricing info */
SELECT
    MAX(price) AS highest_price,
    MIN(price) AS lowest_price,
    ROUND(AVG(price),2) AS avg_price,
    (SELECT price
		FROM   menu_items
		GROUP  BY price
		ORDER  BY COUNT(*) DESC, price DESC
		LIMIT 1) AS most_frequent_price
FROM menu_items;
```
![pricing info](images/pricingInfo.png)

```
/* What are the least expensive items on the menu?*/
SELECT
    item_name,
    category,
    price
FROM
	menu_items
ORDER BY
	price
LIMIT 5;
```
![least expensive](images/leastExpensive.png)
```
/* What are the most expensive items on the menu*/
SELECT
    item_name,
    category,
    price
FROM
	menu_items
ORDER BY
	price DESC
LIMIT 5;
```
![most expensive](images/mostExpensive.png)

```
/* summary pricing info by category */    
SELECT
	category,
    COUNT(menu_item_id) AS num_items,
    MAX(price) AS most_expensive,
    MIN(price) AS least_expensive,
    ROUND(AVG(price),2) AS avg_price
FROM menu_items
GROUP BY category
ORDER BY avg_price DESC, num_items DESC;
```
![category pricing summary](images/categoryPricing.png)

### Orders Table

```
/* view order details table */
SELECT *
FROM
	order_details;
```
![orders table](images/orderTable.png)
```
/* What order dates are included in the table? */
SELECT 
	MIN(order_date) AS first_date,
    MAX(order_date) AS last_date
FROM
	order_details;
 ```
![order date range](images/orderDateRange.png)
```
/* 	How many orders?
	How many items were ordered?
*/

SELECT 
	COUNT(DISTINCT order_id) AS num_orders,
    COUNT(order_details_id) num_order_items
FROM order_details;
```
![order and item counts](images/numOrderItems.png)
```
/*	Which orders had the most items? */
SELECT 
	order_id,
	COUNT(item_id) AS num_items
FROM
	order_details
GROUP BY
	order_id
ORDER BY
	num_items DESC;
```
![most items](images/mostItems.png)
```
/*	How many orders had more than 12 items: */

SELECT COUNT(*) FROM
(SELECT
	order_id,
	COUNT(item_id) AS num_items
FROM
	order_details
GROUP BY
	order_id
HAVING
	num_items > 12) AS num_orders;
```
![orders with more than 12 items](images/twelveOrMore.png)

```
/* Are there any differences in order/item counts by month? */
SELECT 
    MONTH(order_date) AS order_month,
	COUNT(DISTINCT order_id) AS num_orders,
    COUNT(item_id) AS num_items
FROM order_details 
GROUP BY order_month;
```
![order/item counts by month](images/countsByMonth.png)

### Combined Data
