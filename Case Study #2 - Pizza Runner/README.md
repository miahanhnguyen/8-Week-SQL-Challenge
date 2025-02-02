# Case Study #2 - Pizza Runner
[Case Study #2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

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
- Retain the raw tables and create a copy of these tables to perform data cleaning
- Convert all the blank and 'null' value to NULL in SQL
- Standardise the time and distance column in runner_orders to INT type
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
