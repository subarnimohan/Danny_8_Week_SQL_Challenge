# Case Study #1 Danny's Diner
<img src="https://user-images.githubusercontent.com/77920592/199073813-1b1d6aa7-105d-4ab0-9dab-0444b0ef6095.png">

# Introduction
Danny is a fan of Japanese food so in 2021, he decides to visit a cute little restaurant that sells hus three favourite foods: sushi, curry and ramen. 
Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

# Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

# Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

# Information

I have executed the solutions using DB Fiddle using **Schema (PostgreSQL v13)** 

## 1. What is the total amount each customer spent at the restaurant? ##

```` sql
SELECT s.customer_id,CONCAT( '$',sum(m.price)) as price
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
ON s.product_id=m.product_id
GROUP BY s.customer_id 
````
| customer_id | price |
| ----------- | ----- |
| B           | $74   |
| C           | $36   |
| A           | $76   |

---

## 2. How many days has each customer visited the restaurant? ##

```` sql
SELECT s.customer_id,count(distinct s.order_date) visits
FROM dannys_diner.sales s
GROUP BY s.customer_id
ORDER BY visits ASC
````
| customer_id | visits |
| ----------- | ------ |
| C           | 2      |
| A           | 4      |
| B           | 6      |

---

## 3. What was the first item from the menu purchased by each customer?

```` sql
WITH order_ranks AS(
SELECT s.customer_id,s.order_date,s.product_id,m.product_name,
RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) as ranks_order
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
ON s.product_id=m.product_id)

SELECT DISTINCT customer_id,product_name
FROM order_ranks
WHERE ranks_order=1
````

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---
## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```` sql
SELECT product_name,purchased_count

FROM (
SELECT m.product_name, count(*) as purchased_count,RANK() OVER (ORDER BY count(*) DESC) as ranks
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
ON s.product_id=m.product_id
GROUP BY m.product_name) k
WHERE ranks=1

````
| product_name | purchased_count |
| ------------ | --------------- |
| ramen        | 8               |

---
## 5. Which item was the most popular for each customer?

```` sql
WITH top_item AS (
SELECT s.customer_id,m.product_name,COUNT(*) as counts,
RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) as ranks
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
ON s.product_id=m.product_id
GROUP BY s.customer_id,m.product_name
ORDER BY s.customer_id, counts DESC)
SELECT customer_id,product_name,counts
FROM top_item
where ranks=1

````

| customer_id | product_name | counts |
| ----------- | ------------ | ------ |
| A           | ramen        | 3      |
| B           | ramen        | 2      |
| B           | curry        | 2      |
| B           | sushi        | 2      |
| C           | ramen        | 3      |

---

## 6. Which item was purchased first by the customer after they became a member?

```` sql
WITH first_orders AS (
SELECT me.customer_id as cus_id,me.join_date,m.product_name,s.order_date,
RANK() OVER(PARTITION BY me.customer_id ORDER BY s.order_date) as ranks
FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
WHERE s.order_date>=me.join_date)

SELECT cus_id,product_name,order_date
FROM first_orders
WHERE ranks=1
````
| cus_id | product_name | order_date               |
| ------ | ------------ | ------------------------ |
| A      | curry        | 2021-01-07T00:00:00.000Z |
| B      | sushi        | 2021-01-11T00:00:00.000Z |

---

## 7. Which item was purchased just before the customer became a member?

```` sql
WITH orders as (
SELECT me.customer_id as cus_id,me.join_date,m.product_name,s.order_date,
RANK() OVER(PARTITION BY me.customer_id ORDER BY s.order_date DESC) as ranks
FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
WHERE s.order_date<me.join_date)
SELECT cus_id,product_name,order_date,join_date
FROM orders
where ranks=1
````

| cus_id | product_name | order_date               | join_date                |
| ------ | ------------ | ------------------------ | ------------------------ |
| A      | sushi        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| A      | curry        | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z |
| B      | sushi        | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z |

---
## 8. What is the total items and amount spent for each member before they became a member?
```` sql
SELECT me.customer_id as cus_id,count(s.product_id),CONCAT('$',sum(m.price)) as total

FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
WHERE s.order_date<me.join_date
GROUP BY me.customer_id
````  

| cus_id | count | total |
| ------ | ----- | ----- |
| B      | 3     | $40   |
| A      | 2     | $25   |

---

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```` sql
SELECT s.customer_id,
sum(CASE WHEN product_name='sushi' then price*20 ELSE price*10 end) as multiplier

FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
````
| customer_id | multiplier |
| ----------- | ---------- |
| A           | 860        |
| B           | 940        |
| C           | 360        |

---
## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```` sql
SELECT customer_id,sum(points)

FROM (
SELECT s.customer_id,m.product_name,m.price,me.join_date,s.order_date,
s.order_date-me.join_date as diff,
CASE WHEN s.order_date-me.join_date<7 then m.price*20 ELSE m.price*10 end as points
FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
INNER JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
WHERE s.order_date>=me.join_date and s.order_date<='2021-01-31') k
GROUP BY customer_id
````
| customer_id | sum  |
| ----------- | ---- |
| B           | 320  |
| A           | 1020 |

---
## BONUS QUESTION 1: Joining All Things Together

```` sql

SELECT s.customer_id,s.order_date,m.product_name,m.price,
CASE WHEN s.order_date>=me.join_date THEN 'Y'ELSE 'N' end as member

FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
LEFT JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
ORDER BY s.customer_id,s.order_date
````

| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---
## BONUS QUESTION 2: Rank All The Things

```` sql
WITH ranking AS (
SELECT s.customer_id,s.order_date,m.product_name,m.price,
CASE WHEN s.order_date>=me.join_date THEN 'Y'ELSE 'N' end as member

FROM dannys_diner.menu m
INNER JOIN dannys_diner.sales s
ON m.product_id=s.product_id
LEFT JOIN dannys_diner.members me
ON s.customer_id=me.customer_id
ORDER BY s.customer_id,s.order_date)
  
SELECT
customer_id,order_date,product_name,price,member,
  CASE 
  WHEN member='N' then NULL
  ELSE RANK() OVER(PARTITION BY customer_id,member ORDER BY order_date) end as ranking
FROM ranking

````

| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |         |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |         |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |         |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |         |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |         |

---
