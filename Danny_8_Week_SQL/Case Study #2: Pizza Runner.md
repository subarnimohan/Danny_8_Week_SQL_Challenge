# introduction

# Case Study #2 - Pizza Runner #

![image](https://user-images.githubusercontent.com/77920592/199072945-1bdf34ab-bc60-49eb-bcf9-0f21fb9dbe5d.png)

# Overview

The exercise is part of the 8 Week SQL Challenge by Danny Ma. More information on the challenge can be found here: https://8weeksqlchallenge.com/case-study-1/

# Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

# Problem Statement

This case study has LOTS of questions - they are broken up by area of focus including:

Pizza Metrics
Runner and Customer Experience
Ingredient Optimisation
Pricing and Ratings
Bonus DML Challenges (DML = Data Manipulation Language)
Each of the following case study questions can be answered using a single SQL statement.


# Information

I have executed the solutions using DB Fiddle using **Schema (PostgreSQL v13)** 

# Entity Relationship Diagram

# Data Cleaning and Transformation

- From looking into the customer_orders table and the runner_orders table, you can observe that the exclusions and extras columns have 'null' values
- The first step is to replace 'null' values in the columns with '' by creating a temporary table that we would be using in this exercise

<b> Table to clean: customer_orders table </b>

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       | null   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       | null   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |


````sql
CREATE TEMP TABLE customer_orders_temp AS

SELECT order_id,customer_id,pizza_id,

CASE WHEN exclusions IS NULL or exclusions LIKE 'null' then '' else exclusions end as exclusions,
CASE WHEN extras IS NULL or extras='null' then '' ELSE extras end as extras,
order_time
FROM pizza_runner.customer_orders
````

This is the temporary table that has been cleaned

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

---
<b> Table to clean: runner_orders </b>

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

---
The respective datatypes for the columns are also required to be updated

````sql
CREATE TEMP TABLE runner_orders_temp AS

SELECT 
order_id,runner_id,
CASE WHEN pickup_time='null' then '' ELSE pickup_time end as pickup_time,

CASE 
WHEN distance='null' then ''
WHEN distance ILIKE '%km' then trim('km' from distance)
ELSE distance end as distance,

CASE
WHEN duration='null' then '' 
WHEN duration ILIKE '%mins' then trim('mins' from duration)
WHEN duration ILIKE '%minute' then trim('minute' from duration)
WHEN duration ILIKE '%minutes' then trim('minutes' from duration)

ELSE duration end as duration,
CASE WHEN cancellation='null' OR cancellation IS NULL then '' else cancellation end as cancellation
FROM pizza_runner.runner_orders
````
<b> This is the temporary table that has been cleaned </b>

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

--

# A.Pizza Metrics

## How many pizzas were ordered?

````sql
SELECT COUNT(pizza_id)
FROM customer_orders_temp
````
![image](https://github.com/user-attachments/assets/2d8db975-f1b1-4736-ac15-e0aa157d616a)

---
## How many unique customer orders were made
````sql
SELECT COUNT(DISTINCT order_id)
FROM customer_orders_temp
````
![image](https://github.com/user-attachments/assets/29df62f2-361a-43fe-94e0-d91453a67550)
--
## How many successful orders were delivered by each runner?
````sql
SELECT ro.runner_id,COUNT(ro.order_id) as orders
FROM runner_orders_temp ro
WHERE ro.cancellation NOT ILIKE'%cancellation%'
GROUP BY ro.runner_id
````

![image](https://github.com/user-attachments/assets/f3e11b23-8b52-4290-b81b-9bbb33b7868b)

--
## How many of each type of pizza was delivered?


````sql
SELECT cc.pizza_id,count(*)
FROM runner_orders_temp r
INNER JOIN customer_orders cc
ON r.order_id=cc.order_id
WHERE cancellation NOT ILIKE '%cancellation%'
GROUP BY cc.pizza_id
````

![image](https://github.com/user-attachments/assets/102e2d9c-75b9-444d-9e7a-cae7a420d049)

--
## How many Vegetarian and Meatlovers were ordered by each customer?

````sql
SELECT customer_id,pizza_name,count(*)
	FROM pizza_names pn

INNER JOIN customer_orders_temp cc
	ON pn.pizza_id=cc.pizza_id
GROUP BY customer_id,pizza_name
ORDER BY customer_id
````
--

![image](https://github.com/user-attachments/assets/741c1c23-fd4e-470f-9b21-2cb36b21b90c)

## What was the maximum number of pizzas delivered in a single order?

````sql
WITH ranks AS (
SELECT r.order_id,count(cc.pizza_id) as pizzas,RANK() OVER(ORDER BY count(cc.pizza_id) DESC)
FROM runner_orders_temp r
INNER JOIN customer_orders_temp cc
ON r.order_id=cc.order_id
WHERE cancellation NOT LIKE '%Cancellation%'
GROUP BY r.order_id
ORDER BY pizzas DESC)

SELECT order_id,pizzas
FROM ranks
where rank=1
````
![image](https://github.com/user-attachments/assets/a8144955-d125-4e3e-be98-f36da28a1036)

#### <b> The maximum number of pizzas delivered in a single order is 3
--

## For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql

SELECT cc.customer_id,
	
	sum(CASE WHEN cc.exclusions!='' OR cc.extras!='' then 1 else 0 end) as at_least_one,
	sum(CASE WHEN cc.exclusions='' AND cc.extras='' then 1 else 0 end) as no_change
FROM runner_orders_temp r
	
	INNER JOIN customer_orders_temp cc
	
ON r.order_id=cc.order_id
WHERE cancellation NOT LIKE '%Cancellation%'
GROUP BY cc.customer_id
ORDER BY cc.customer_id

````
--
![image](https://github.com/user-attachments/assets/3557538a-38db-4cd3-81aa-25e6c073832e)

## How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT 
	
SUM(CASE WHEN cc.exclusions!='' AND cc.extras!='' then 1 else 0 end) as at_least_one
FROM runner_orders_temp r
	
INNER JOIN customer_orders_temp cc
	
ON r.order_id=cc.order_id
WHERE cancellation NOT LIKE '%Cancellation%'
````

![image](https://github.com/user-attachments/assets/038af6b2-ec7b-4776-9f46-35e7d2aeb516)

## What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT EXTRACT(hour from order_time),count(order_id)
FROM customer_orders_temp cc
GROUP BY EXTRACT(hour from order_time)

````
![image](https://github.com/user-attachments/assets/38a6a11e-8fa6-4737-bfe6-11e9ead78c5c)
--
## What was the volume of orders for each day of the week?

````sql
SELECT TO_CHAR(cc.order_time,'day'),count(cc.order_id)

FROM customer_orders_temp cc
GROUP BY TO_CHAR(cc.order_time,'day')
````
![image](https://github.com/user-attachments/assets/710a4fb0-dcf0-4723-bc47-89489bdec344)

--
# B Runner and Customer Experience

## How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql

````
--

````sql
````
--

````sql
````
--

````sql
````
--



