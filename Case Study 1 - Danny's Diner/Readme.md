# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Solution](#solution)

All the information about the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Solution

Any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/huutrung16/). I'm using MySQL for the whole Case Study.

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT s.customer_id, 
   SUM(m.price) AS total_sales
FROM sales s
INNER JOIN  menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
````

#### Steps:
- Use **JOIN** to merge `sales` and `menu` tables as `sales.customer_id` and `menu.price` are from both tables.
- Use **SUM** to calculate the total sales contributed by each customer.
- Group the aggregated results by `sales.customer_id`. 

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT customer_id, 
   COUNT(DISTINCT(order_date)) AS times_visited
FROM sales
GROUP BY customer_id;
````

#### Steps:
- To determine the unique number of visits for each customer, utilize **COUNT(DISTINCT `order_date`)**.

#### Answer:
| customer_id | times_visited |
| ----------- | -----------   |
| A           | 4             |
| B           | 6             |
| C           | 2             |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH cte AS (
   SELECT s.customer_id, 
      s.product_id, m.product_name,
      DENSE_RANK() OVER(
         PARTITION BY s.customer_id
	 ORDER BY s.order_date) AS order_rank
   FROM sales s
   INNER JOIN menu m
   ON s.product_id = m.product_id
)
SELECT customer_id,
   product_name
FROM cte
WHERE order_rank = 1
GROUP BY customer_id, product_name
;
````

#### Steps:
- Create a (CTE) named `cte`. Within the CTE, create a new column `order_rank` and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by `customer_id`, and the **ORDER BY** clause orders the rows within each partition by `order_date`.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, which represents the first row within each `customer_id` partition.
- Use the GROUP BY clause to group the result by `customer_id` and `product_name`.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT COUNT(sales.product_id) AS most_purchased, 
   menu.product_name
FROM sales
INNER JOIN menu
ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased DESC
LIMIT 1
;
````

#### Steps:
- Perform a **COUNT** aggregation on the `sales.product_id` column and **ORDER BY** the result in descending order using `most_purchased` field.
- Apply the **LIMIT** 1 clause to filter and retrieve the highest number of purchased items.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Most purchased item on the menu is ramen which is 8 times.

***

### 5. Which item was the most popular for each customer?

````sql
WITH cte AS(
   SELECT sales.customer_id, 
      menu.product_name, 
      COUNT(menu.product_id) AS order_count,
      DENSE_RANK() OVER(
         PARTITION BY sales.customer_id
         ORDER BY COUNT(sales.product_id) DESC) AS ranked
   FROM sales
   INNER JOIN menu
   ON sales.product_id = menu.product_id
   GROUP BY sales.customer_id, menu.product_name
)
SELECT customer_id, 
   product_name, 
   order_count
FROM cte
WHERE ranked = 1
;
````

*Each user may have more than 1 favourite item.*

#### Steps:
- Create a CTE named `cte` and within the CTE, join the `sales` table and `menu` table using the `product_id` column.
- Calculate the count of `menu.product_id` occurrences for each group. 
- Utilize the **DENSE_RANK()** window function to calculate the ranking of each `sales.customer_id` partition based on the count of orders **COUNT(`sales.product_id`)** in descending order.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the highest order count for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu.

***

### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH cte AS(
   SELECT members.customer_id, 
      sales.product_id,
      ROW_NUMBER() OVER(
         PARTITION BY members.customer_id
         ORDER BY sales.order_date ASC)
         AS row_num
   FROM members
   INNER JOIN sales
   ON members.customer_id = sales.customer_id AND members.join_date < sales.order_date
)
SELECT menu.product_name, 
   cte.customer_id
FROM cte
INNER JOIN menu
ON cte.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC
;
```

#### Steps:
- Create a CTE named `cte` and within the CTE, select the appropriate columns and calculate the row number using the **ROW_NUMBER()** window function. The **PARTITION BY** clause divides the data by `members.customer_id` and the **ORDER BY** clause orders the rows by `sales.order_date`.
- Join tables `members` and `sales` on `customer_id` column. Apply a condition to only include sales that occurred *after* the member's `join_date`.
- In the outer query, join the `cte` with the `menu` on the `product_id` column.
- In the **WHERE** clause, filter to retrieve only the rows where the row_num column equals 1, representing the first row within each `customer_id` partition.
- Order result by `customer_id` in ascending order.

#### Answer:
| product_name | customer_id |
| ----------- | ---------- |
| ramen       | A        |
| sushi       | B       |

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH cte AS(
   SELECT sales.customer_id, 
      sales.order_date, 
      sales.product_id,
      ROW_NUMBER() OVER(
         PARTITION BY sales.customer_id
         ORDER BY sales.order_date DESC) AS row_num
   FROM sales
   INNER JOIN members
   ON sales.customer_id = members.customer_id AND sales.order_date < members.join_date
)
SELECT cte.customer_id, 
   menu.product_name
FROM cte
INNER JOIN menu
ON cte.product_id = menu.product_id
WHERE row_num = 1
ORDER BY cte.customer_id ASC
;
````

#### Steps:
- Create a CTE called `cte`. In the CTE, calculate the rank using the **ROW_NUMBER()** window function. The rank is determined based on the order dates of the sales in descending order within each customer's group.
- Join `members` table with `sales` table based on the `customer_id` column, ensuring the orders that occurred *before* the customer joined as a member.
- Join `cte` with `menu` table based on `product_id` column.
- Filter the result set to include only the rows where the rank is 1.
- Sort the result by `customer_id` in ascending order.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |

- Both customers' last order before becoming members are sushi.

***

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT members.customer_id, 
   COUNT(DISTINCT(sales.product_id)) AS total_items, 
   SUM(menu.price) AS total_amount
FROM sales
INNER JOIN members
ON sales.customer_id = members.customer_id AND sales.order_date < members.join_date
INNER JOIN menu
ON sales.product_id = menu.product_id
GROUP BY members.customer_id
ORDER BY members.customer_id
;
```

#### Steps:
- Select the columns `members.customer_id` and calculate the count of `sales.product_id` as total_items for each customer and the sum of `menu.price` as total_amount.
- From `sales` table, join `members` table on `customer_id` column, ensuring that `sales.order_date` is earlier than `members.join_date`.
- Then, join `menu` table to `sales` table on `product_id` column.
- Group the results by `members.customer_id`.
- Order the result by `members.customer_id`.

#### Answer:
| customer_id | total_items | total_amount |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

```sql
WITH cte AS(
   SELECT sales.customer_id, 
      menu.product_name,
      CASE
         WHEN menu.product_name = 'sushi' THEN 20
         ELSE 10
         END AS points
   FROM sales
   INNER JOIN menu
   ON sales.product_id = menu.product_id
)
SELECT cte.customer_id, 
   SUM(cte.points*menu.price) AS personal_points
FROM cte
INNER JOIN menu
ON cte.product_name = menu.product_name
GROUP BY cte.customer_id
;
```

#### Steps:
- Create a CTE called `cte`. In `cte`, calculate the `points` for each `product_name`. Each $1 spent = 10 points. However, `product_name` sushi gets 2x points, so each $1 spent on sushi = 20 points.
- Use conditional CASE statement:
	- If `product_name` = 'sushi', then 20 (points).
	- Otherwise, multiply 10 (points).
- Then, `JOIN` `cte` with menu and calculate the total points for each customer using `SUM cte.points*menu.price`.

#### Answer:
| customer_id | personal_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

```sql
WITH cte AS(
   SELECT sales.customer_id, 
      menu.product_id,
      sales.order_date,
      members.join_date,
      sales.order_date - members.join_date AS differ,
      menu.price,
      menu.product_name,
      CASE
         WHEN menu.product_name = 'sushi' THEN 20
         WHEN MONTH(sales.order_date) > 1 THEN 0
         WHEN sales.order_date - members.join_date < 0 THEN 10
         WHEN 0 <= sales.order_date - members.join_date AND sales.order_date - members.join_date < 7 THEN 20
         WHEN sales.order_date - members.join_date >= 7 THEN 10
         END AS points
   FROM sales
   INNER JOIN menu
   ON sales.product_id = menu.product_id
   INNER JOIN members
   ON sales.customer_id = members.customer_id
   ORDER BY sales.customer_id, sales.order_date
)
SELECT cte.customer_id,
   SUM(cte.points * menu.price) AS personal_points
FROM cte
INNER JOIN menu
ON cte.product_id = menu.product_id
GROUP BY cte.customer_id
ORDER BY cte.customer_id
;
```

#### Steps:
- Create a CTE called `cte`. In `cte`, calculate the `points` for each `product_name`.
- Use conditional `CASE` statement:
	- `product_name` sushi still gets 2x points even if the `orders_date` is not qualified.
	- _"At the end of the January"_ means that just the orders in January get points.
	- There are two cases if the order is not in the first week from the `join_date`. First, the `order_date` < the `join_date`. Second, the `order_date` >= the 		`join_date` + 7. Then, every order in these 2 cases will get 10 points (exception for sushi).
	- In the first week from the `join_date`, every order will get 20 points.
- Then, join `menu` table based on the `product_id` column.
- In the outer query, `JOIN` `cte` with `menu` and do `SUM` to calculate the `personal_points`.

#### Answer:
| customer_id | personal_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***

## BONUS QUESTIONS

### Join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

```sql
SELECT sales.customer_id, 
   sales.order_date,
   menu.product_name,
   menu.price,
   CASE
      WHEN sales.order_date < members.join_date THEN 'N'
      WHEN sales.order_date >= members.join_date THEN 'Y'
      ELSE 'N'
      END AS member_status
FROM sales
LEFT JOIN members
ON sales.customer_id = members.customer_id
INNER JOIN menu
ON sales.product_id = menu.product_id
;
```
 
#### Answer: 
| customer_id | order_date | product_name | price | member_status |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***

### Rank All The Things - Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.

```sql
WITH cte AS(
   SELECT sales.customer_id, 
      sales.order_date,
      menu.product_name,
      menu.price,
      CASE
         WHEN sales.order_date < members.join_date THEN 'N'
	 WHEN sales.order_date >= members.join_date THEN 'Y'
         ELSE 'N'
	 END AS member_status
   FROM sales
   LEFT JOIN members
   ON sales.customer_id = members.customer_id
   INNER JOIN menu
   ON sales.product_id = menu.product_id
)
SELECT *,
   CASE
      WHEN member_status = 'N' THEN 'NULL'
      WHEN member_status = 'Y' THEN DENSE_RANK() OVER(
				      PARTITION BY cte.customer_id, member_status
                                      ORDER BY cte.order_date ASC
      )
      END AS ranking
FROM cte
;
```

#### Answer: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

***
