# Case Study #2 - Pizza Runner
[Case Study #2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

![image](https://github.com/user-attachments/assets/445ea183-9a61-473d-995e-299878f3e1d8)


## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/0f2e58ad-7bfb-4d29-b1db-ec978f8a49c1)

This case study has 5 focus areas listed below

## Table of contents
A. Pizza Metrics  
B. Runner and Customer Experience  
C. Ingredient Optimisation  
D. Pricing and Ratings  
E. Bonus DML Challenges (DML = Data Manipulation Language)  

## Data cleaning
Create a cleaned table for customer_orders and runner_orders
````sql
CREATE TABLE cleaned_customer_orders AS
SELECT order_id, customer_id, pizza_id,
CASE
	WHEN exclusions = 'null' or exclusions = '' THEN NULL
    ELSE exclusions
END AS exclusions,
CASE
	WHEN extras = 'null' or extras = '' THEN NULL
    ELSE extras
END AS extras,
order_time
FROM customer_orders;

CREATE TABLE cleaned_runner_orders AS
SELECT order_id, runner_id,
NULLIF(pickup_time,'null')::TIMESTAMP AS pickup_time,
CASE
    WHEN distance = 'null' THEN NULL
    WHEN distance LIKE '%km' THEN 
    CAST(
      REGEXP_REPLACE(distance, '\D+', '', 'g') AS FLOAT
      )
    ELSE CAST(distance AS FLOAT) -- Using CAST to type cast the data to FLOAT
END AS distance,
CASE
    WHEN duration = 'null' THEN NULL
    WHEN duration LIKE '%min%' THEN 
        CAST(
            REGEXP_REPLACE(duration, '\D+', '', 'g') AS INTEGER
        )
    ELSE 
        CAST(duration AS INTEGER)
END AS duration,
CASE
	WHEN cancellation = 'null' or cancellation = '' THEN NULL
    ELSE cancellation
END AS cancellation
FROM runner_orders;
````
