https://8weeksqlchallenge.com/case-study-3/

https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16

## A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

```sql
SELECT 
  s.customer_id,
  s.start_date,
  p.plan_name
FROM foodie_fi.subscriptions as s
LEFT JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id, s.start_date
```

| customer_id | start_date | plan_name     |
| ----------- | ---------- | ------------- |
| 1           | 2020-08-01 | trial         |
| 1           | 2020-08-08 | basic monthly |
| 2           | 2020-09-20 | trial         |
| 2           | 2020-09-27 | pro annual    |
| 11          | 2020-11-19 | trial         |
| 11          | 2020-11-26 | churn         |
| 13          | 2020-12-15 | trial         |
| 13          | 2020-12-22 | basic monthly |
| 13          | 2021-03-29 | pro monthly   |
| 15          | 2020-03-17 | trial         |
| 15          | 2020-03-24 | pro monthly   |
| 15          | 2020-04-29 | churn         |
| 16          | 2020-05-31 | trial         |
| 16          | 2020-06-07 | basic monthly |
| 16          | 2020-10-21 | pro annual    |
| 18          | 2020-07-06 | trial         |
| 18          | 2020-07-13 | pro monthly   |
| 19          | 2020-06-22 | trial         |
| 19          | 2020-06-29 | pro monthly   |
| 19          | 2020-08-29 | pro annual    |

Customer 1 : This customer subscribed trial plan on 1st August 2020 and after the trial ended, this customer subscribed to basic monthly on 8 August 2020.

Customer 2 : This customer subscribed trial plan on 20th September 2020, then subscribed to pro annual on 27th September 2020.

Customer 11 : This customer subscribed trial plan on 19th November 2020, then this customer churn the plan on 26th November 2020.

Customer 13 : This customer subscribed trial plan on 15th December 2020 and after the trial ended, this customer subscribed to basic monthly on 22 December 2020. Once the customer find this app is suitable to the needs, then this customer subscribed to pro monthly on 29th March 2020.

Customer 15 : This customer subscribed trial plan on 17th March 2020, then subscribed to pro manual on 24th March 2020. Then, this customer churn the plan on 29th April 2020.

Customer 16 : This customer subscribed trial plan on 31st May 2020 and after the trial ended, this customer subscribed to basic monthly on 7th June 2020. Once the customer find this app is suitable to the needs, then this customer subscribed to pro annual on 21st October 2020.

Customer 18 : This customer subscribed trial plan on 6th July 2020, then subscribed to pro monthly on 13th July 2020.

Customer 19 : This customer subscribed trial plan on 22nd June 2020 and after the trial ended, this customer subscribed to pro monthly on 29th June 2020. Once the customer find this app is suitable to the needs, then this customer subscribed to pro monthly on 29th August 2020.

---
## B. Data Analysis Questions
1. How many customers has Foodie-Fi ever had?
```sql
SELECT 
  COUNT (DISTINCT s.customer_ID) as total_customers
FROM foodie_fi.subscriptions as s
LEFT JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id
```
| total_customers |
| --------------- |
| 1000            |
---

2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT 
  EXTRACT (MONTH from s.start_date) AS month,
  COUNT (*) as total 
FROM foodie_fi.subscriptions as s
LEFT JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id
WHERE s.plan_id = 0
GROUP BY EXTRACT (MONTH from s.start_date)
ORDER BY EXTRACT (MONTH from s.start_date)
```
| month | total |
| ----- | ----- |
| 1     | 88    |
| 2     | 68    |
| 3     | 94    |
| 4     | 81    |
| 5     | 88    |
| 6     | 79    |
| 7     | 89    |
| 8     | 88    |
| 9     | 87    |
| 10    | 79    |
| 11    | 75    |
| 12    | 84    |

---


3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
 
```sql
SELECT 
  p.plan_id,
  p.plan_name,
COUNT (*) as total 
FROM foodie_fi.subscriptions as s
LEFT JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id
WHERE EXTRACT (YEAR from s.start_date) > 2020
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id
```
| plan_id | plan_name     | total |
| ------- | ------------- | ----- |
| 1       | basic monthly | 8     |
| 2       | pro monthly   | 60    |
| 3       | pro annual    | 63    |
| 4       | churn         | 71    |

---

4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT
total_churn,
ROUND (100.0 * total_churn/total_customer,1) as per_churn
FROM (
SELECT
SUM 
(CASE 
 	WHEN p.plan_id = 4 THEN 1
	ELSE 0
	END
) as total_churn,
COUNT (DISTINCT s.customer_id) as total_customer
FROM foodie_fi.subscriptions as s
LEFT JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id)t
```

| total_churn | per_churn |
| ----------- | --------- |
| 307         | 30.7      |

---

5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH CTE_churn_rank AS (
	SELECT
		s.customer_id,
		p.plan_name,
	CASE 
		WHEN p.plan_name = 'churn' AND rank () OVER (PARTITION BY s.customer_id ORDER BY s.start_date) = 2 THEN 1
		ELSE 0
	END as total_churn_after_trial
	FROM foodie_fi.subscriptions as s
	LEFT JOIN foodie_fi.plans as p
	ON s.plan_id = p.plan_id
)


SELECT 
	total_churn_trial,
	(100 * total_churn_trial/total_customer) as per_churn_trial
FROM (
	SELECT 
		COUNT (DISTINCT customer_id) as total_customer,
		SUM (total_churn_after_trial) as total_churn_trial
	FROM CTE_churn_rank)t
```

| total_churn_trial | per_churn_trial |
| ----------------- | --------------- |
| 92                | 9               |

---
7. What is the number and percentage of customer plans after their initial free trial?
8. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
9. How many customers have upgraded to an annual plan in 2020?
10. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
11. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
12. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)
