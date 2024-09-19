# Introduction

# Case Study #3 - Foodie-Fi
<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

# Overview

The exercise is part of the 8 Week SQL Challenge by Danny Ma. More information on the challenge can be found here: https://8weeksqlchallenge.com/case-study-1/

# Introduction

Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

# Problem Statement
<add text>
  
# Information

I have executed the solutions using DB Fiddle using **Schema (PostgreSQL v13)** 

# Entity Relationship Diagram
<add diagram>
# Case Study

This case study is split into an initial data understanding question before diving straight into data analysis questions before finishing with 1 single extension challenge

# Data Analysis

## 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT (customer_id))
FROM subscriptions
```
![image](https://github.com/user-attachments/assets/9f714216-19a1-441e-a98e-0a7069ec4b6e)
---

## 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT EXTRACT(month from start_date),count(customer_id) as counts
FROM plans pp
INNER JOIN subscriptions ss
ON pp.plan_id=ss.plan_id
WHERE pp.plan_id=0
GROUP BY EXTRACT(month from start_date)
ORDER BY counts ASC
```
![image](https://github.com/user-attachments/assets/32bba21f-875d-496e-a998-be15b2d8277c)

---
## 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
SELECT plan_name,count(*)
FROM plans pp
INNER JOIN subscriptions ss
ON pp.plan_id=ss.plan_id
WHERE EXTRACT(YEAR FROM start_date)>2020
GROUP BY plan_name
```
![image](https://github.com/user-attachments/assets/1378fbfb-fac3-460c-8455-d37a198ce9d6)

---
## 4.What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
SELECT
	COUNT(DISTINCT ss.customer_id),
	ROUND(100*COUNT(DISTINCT ss.customer_id)::decimal/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) as subscriptions
	
FROM plans pp
INNER JOIN subscriptions ss
ON pp.plan_id=ss.plan_id
WHERE plan_name='churn'
```
![image](https://github.com/user-attachments/assets/cc61e56f-803d-4c60-b5c4-4af70ec872b7)

---

## 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH c as (SELECT
	customer_id,ss.plan_id,plan_name,start_date,
	RANK() OVER(PARTITION BY customer_id ORDER BY start_date) as ranks
	
FROM plans pp
INNER JOIN subscriptions ss
ON pp.plan_id=ss.plan_id)
	
SELECT COUNT(CASE WHEN plan_name='churn' and ranks=2 then 'churned right after' else null end) as churn_counts,

	ROUND(100*COUNT(CASE WHEN plan_name='churn' and ranks=2 then 'churned right after' else null end)::decimal/COUNT(DISTINCT customer_id),1) as churn_pct	
FROM c
```
![image](https://github.com/user-attachments/assets/27b6c37a-6a07-431b-a2bd-523094cb0707)

---
## 6. What is the number and percentage of customer plans after their initial free trial?
```sql
WITH tt as (SELECT plan_id,customer_id,start_date, LEAD(plan_id,1) OVER(PARTITION BY customer_id ORDER BY plan_id) as plan_new	
FROM subscriptions ss)

SELECT plan_new,
	COUNT(*),
	ROUND(100*COUNT(customer_id)::DECIMAL/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1)
FROM tt
WHERE plan_new IS NOT NULL
GROUP BY plan_new
```
---

```sql
```
---

```sql
```
---


```sql
```
---


```sql
```
---

```sql
```
---

