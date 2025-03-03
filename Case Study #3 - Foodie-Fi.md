# Case Study #3 - Foodie-Fi
[Case Study #3 - Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/fcf6fee1-a76f-401c-a9a1-81b8f930eb83)

## A. A brief description about each customer’s onboarding journey based off the first 8 sample customers.
- Customer usually started with the trial plan which was free (price = 0.00)
- After the trial period (7 days), majority of customers upgraded to basic monthly plan ($9.90)
- 1 customer in this summary (id = 2) upgraded to pro annual right after the trial period ($199.00)
- 1 customer (id = 8) upgraded to pro monthly (19.90) 2 months after basic monthly plan

````sql
    SELECT s.customer_id, s.start_date, p.plan_name, p.price
    FROM subscriptions s
    JOIN plans p
    	ON s.plan_id = p.plan_id
    WHERE s.customer_id <= 8
    ORDER BY s.customer_id, s.start_date;
````
| customer_id | start_date | plan_name     | price  |
| ----------- | ---------- | ------------- | ------ |
| 1           | 2020-08-01 | trial         | 0.00   |
| 1           | 2020-08-08 | basic monthly | 9.90   |
| 2           | 2020-09-20 | trial         | 0.00   |
| 2           | 2020-09-27 | pro annual    | 199.00 |
| 3           | 2020-01-13 | trial         | 0.00   |
| 3           | 2020-01-20 | basic monthly | 9.90   |
| 4           | 2020-01-17 | trial         | 0.00   |
| 4           | 2020-01-24 | basic monthly | 9.90   |
| 4           | 2020-04-21 | churn         |        |
| 5           | 2020-08-03 | trial         | 0.00   |
| 5           | 2020-08-10 | basic monthly | 9.90   |
| 6           | 2020-12-23 | trial         | 0.00   |
| 6           | 2020-12-30 | basic monthly | 9.90   |
| 6           | 2021-02-26 | churn         |        |
| 7           | 2020-02-05 | trial         | 0.00   |
| 7           | 2020-02-12 | basic monthly | 9.90   |
| 7           | 2020-05-22 | pro monthly   | 19.90  |
| 8           | 2020-06-11 | trial         | 0.00   |
| 8           | 2020-06-18 | basic monthly | 9.90   |
| 8           | 2020-08-03 | pro monthly   | 19.90  |

## B. Data Analysis  
**1. How many customers has Foodie-Fi ever had?**
````sql
    SELECT count(distinct customer_id)
    FROM subscriptions;
````
| count |
| ----- |
| 1000  |

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**
````sql
    SELECT date_trunc('month', start_date) as month_start,
    count(*) as trial_count
    FROM subscriptions
    WHERE plan_id = 0
    GROUP BY month_start
    ORDER BY month_start;
````
| month_start            | trial_count |
| ---------------------- | ----------- |
| 2020-01-01 00:00:00+00 | 88          |
| 2020-02-01 00:00:00+00 | 68          |
| 2020-03-01 00:00:00+00 | 94          |
| 2020-04-01 00:00:00+00 | 81          |
| 2020-05-01 00:00:00+00 | 88          |
| 2020-06-01 00:00:00+00 | 79          |
| 2020-07-01 00:00:00+00 | 89          |
| 2020-08-01 00:00:00+00 | 88          |
| 2020-09-01 00:00:00+00 | 87          |
| 2020-10-01 00:00:00+00 | 79          |
| 2020-11-01 00:00:00+00 | 75          |
| 2020-12-01 00:00:00+00 | 84          |

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**
````sql
    SELECT p.plan_name, count (p.plan_name) as count_plan
    FROM plans p
    JOIN subscriptions s
    	ON p.plan_id = s.plan_id
    WHERE start_date >= '2021-01-01'
    GROUP BY p.plan_name;
````

| plan_name     | count_plan |
| ------------- | ---------- |
| pro annual    | 63         |
| churn         | 71         |
| pro monthly   | 60         |
| basic monthly | 8          |

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
````sql
    WITH churn_customer AS (
      SELECT count(distinct customer_id) as churn_count
      FROM subscriptions
      WHERE plan_id = 4
      ),
      total_customer AS (
      SELECT count(distinct customer_id) as total_count
      FROM subscriptions
      )
    
    SELECT c.churn_count as customer_count,
    	ROUND(c.churn_count::DECIMAL/t.total_count * 100, 1) as customer_percentage
    FROM churn_customer c
    CROSS JOIN total_customer t;
````

| customer_count | customer_percentage |
| -------------- | ------------------- |
| 307            | 30.7                |

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
- Using self-join to separate plan_id 0 and 4 into 2 separate columns and join on customer_id
- Filter out a table with customer starts with trial followed by churn immediately
- Use a middle table s3 subscription to ensure that there is no start_date in between the trial and churn date (using NOT EXISTS)
````sql
    WITH trial_churn_filter AS (
    SELECT s1.customer_id
    FROM subscriptions s1
    JOIN subscriptions s2
    ON s1.customer_id = s2.customer_id
    WHERE s1.plan_id = 0 and s2.plan_id = 4 -- Extract customer_id with both trial and churn date
    AND s1.start_date < s2.start_date -- Churn date (4) has be after trial date (0)
    AND NOT EXISTS -- Check no other plans exist between trial (0) and churn (4) using a middle table
    (SELECT 1
     FROM subscriptions s3
     WHERE s3.customer_id = s1.customer_id
     AND s3.start_date < s2.start_date
     AND s3.start_date > s1.start_date)
    ),
    total_customer AS (
    SELECT count(distinct customer_id) as total_count
    FROM subscriptions
    ),
    churn_customer AS (
    SELECT count(distinct customer_id) as churn_count
    FROM trial_churn_filter
    )
    
    SELECT cc.churn_count as customer_count,
    ROUND(cc.churn_count::decimal/tc.total_count * 100, 0) as customer_percentage
    FROM churn_customer cc
    CROSS JOIN total_customer tc;
````
| customer_count | customer_percentage |
| -------------- | ------------------- |
| 92             | 9                   |

**6. What is the number and percentage of customer plans after their initial free trial?**
- Filter out the customer plans followed by a trial using self-join
- Calculate the total number of customer from this filter table and percentage of each plan of this filter table
````sql
    WITH plan_filter AS (
    SELECT s1.customer_id, s2.plan_id
    FROM subscriptions s1
    JOIN subscriptions s2
    ON s1.customer_id = s2.customer_id
    WHERE s1.plan_id = 0 and s2.plan_id != 0
    AND s1.start_date < s2.start_date
    ),
    total_customer AS (
    SELECT count(distinct customer_id) as total_count
    FROM plan_filter
    ),
    plan_customer AS (
    SELECT p.plan_id, p.plan_name, count(distinct customer_id) as plan_count
    FROM plan_filter pf
    JOIN plans p
      ON pf.plan_id = p.plan_id
    GROUP BY p.plan_name, p.plan_id
    ORDER BY p.plan_id
    )
    
    SELECT pc.plan_name, pc.plan_count,
    ROUND(pc.plan_count::decimal/tc.total_count * 100, 0) as plan_percentage
    FROM plan_customer pc
    CROSS JOIN total_customer tc;
````

| plan_name     | plan_count | plan_percentage |
| ------------- | ---------- | --------------- |
| basic monthly | 546        | 55              |
| pro monthly   | 539        | 54              |
| pro annual    | 258        | 26              |
| churn         | 307        | 31              |

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
- Filter out customer plans before 2020-12-31. Since each customer can have multiple plans before that day (for example: trial followed by basic monthly etc.), we need to filter out the latest plan for each customer only
- To do that, after filtering plans before 2020-12-31, using self-join to exclude all the plans before the latest plan filter table (hence using NOT EXISTS)
- If we don't do this filter, the count and percentage breakdown will be inflated as one customer can have multiple plans.
````sql
    WITH latest_plan AS (
    SELECT s.customer_id, s.plan_id
    FROM subscriptions s
    WHERE s.start_date <= '2020-12-31'
    AND NOT EXISTS ( -- This condition is to ensure that each customer is filtered with the latest plan
      SELECT 1
      FROM subscriptions s1
      WHERE s1.customer_id = s.customer_id
      AND s1.start_date <= '2020-12-31'
      AND s1.start_date > s.start_date
      )
    ),
    total_customer AS (
    SELECT count(distinct customer_id) as total_count
    FROM latest_plan
    ),
    plan_customer AS (
    SELECT p.plan_id, p.plan_name, count(distinct customer_id) as plan_count
    FROM latest_plan lp
    JOIN plans p
      ON lp.plan_id = p.plan_id
    GROUP BY p.plan_name, p.plan_id
    ORDER BY p.plan_id
    )
    
    SELECT pc.plan_name, pc.plan_count,
    round(pc.plan_count::decimal/tc.total_count * 100, 0) as plan_percentage
    FROM plan_customer pc
    CROSS JOIN total_customer tc;
````

| plan_name     | plan_count | plan_percentage |
| ------------- | ---------- | --------------- |
| trial         | 19         | 2               |
| basic monthly | 224        | 22              |
| pro monthly   | 326        | 33              |
| pro annual    | 195        | 20              |
| churn         | 236        | 24              |
