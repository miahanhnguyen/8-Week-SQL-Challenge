# Case Study #4 - Data Bank
[Case Study #4 - Data Bank](https://8weeksqlchallenge.com/case-study-4/)

<img src="https://user-images.githubusercontent.com/81607668/130343294-a8dcceb7-b6c3-4006-8ad2-fab2f6905258.png" alt="Image" width="500" height="520">

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f9f41b1c-50c3-4961-841c-48c247db2285)

## A. Customer Nodes Exploration
**1. How many unique nodes are there on the Data Bank system?**
````sql
    SELECT count(distinct node_id) as number_node
    FROM customer_nodes;
````
| number_node |
| ----------- |
| 5           |

**2. What is the number of nodes per region?**
````sql
    SELECT distinct region_id, count(distinct node_id) as number_node
    FROM customer_nodes
    GROUP BY region_id;
````
| region_id | number_node |
| --------- | ----------- |
| 1         | 5           |
| 4         | 5           |
| 5         | 5           |
| 3         | 5           |
| 2         | 5           |

**3. How many customers are allocated to each region?**
````sql
    SELECT distinct region_id, count(distinct customer_id) as number_customer
    FROM customer_nodes
    GROUP BY region_id;
````
| region_id | number_customer |
| --------- | --------------- |
| 1         | 110             |
| 3         | 102             |
| 4         | 95              |
| 2         | 105             |
| 5         | 88              |

**4. How many days on average are customers reallocated to a different node?**
````sql
    SELECT avg(end_date - start_date) as avg_reallocation
    FROM customer_nodes;
````
| avg_reallocation    |
| ------------------- |
| 416373.411714285714 |

**Comment:**
- This number seems very unrealistic (416373 days ~ 1140.75 years)
- Check MIN and MAX start_date and end_date to see the recorded range in the database: '9999-12-31' is the MAX end_date
- Exclude entries with this end_date and re-calculate the average
````sql
select avg(end_date - start_date)
from customer_nodes
where end_date != '9999-12-31';
````
| avg_reallocation    |
| ------------------- |
| 14.6340000000000000 |

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**
````sql
    select region_id,
    percentile_cont (0.05) within group (order by(end_date - start_date)) as median,
    percentile_cont (0.8) within group (order by(end_date - start_date)) as percentile_80th,
    percentile_cont (0.95) within group (order by(end_date - start_date)) as percentile_95th
    from customer_nodes
    where end_date != '9999-12-31'
    group by region_id;
````
| region_id | median | percentile_80th | percentile_95th |
| --------- | ------ | --------------- | --------------- |
| 1         | 1      | 23              | 28              |
| 2         | 1      | 23              | 28              |
| 3         | 1      | 24              | 28              |
| 4         | 1      | 23              | 28              |
| 5         | 1      | 24              | 28              |

## B. Customer Transactions
**1. What is the unique count and total amount for each transaction type?**
````sql
select distinct txn_type, sum(txn_amount)
from customer_transactions
group by txn_type;
````
|txn_type	|sum     |
|-----------|--------|
|purchase	|806537  |
|deposit	|1359168 |
|withdrawal	|793003  |

**2. What is the average total historical deposit counts and amounts for all customers?**
````sql
SELECT 
    COUNT(*) AS total_deposits,            -- Counts all deposit transactions
    AVG(txn_amount) AS avg_deposit_amount  -- Averages deposit amounts
FROM customer_transactions
WHERE txn_type = 'deposit';
````
|total_deposits	|avg_deposit_amount  |
|---------------|--------------------|
|2671	        |508.8611007113440659|

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
- Create a CTE to mark customers who make deposit, purchase and withdrawal, sum the total of those, grouped by each customer_id
- With the 3 new columns: deposit_count, purchase_count, withdrawal_count, we can then filter out who make more than 1 deposit and at least 1 purchase or 1 withdrawal each month
````sql
WITH customer_activity AS (
  SELECT customer_id,
  DATE_TRUNC('month', txn_date)::date as month,
  SUM (
    CASE
    WHEN txn_type = 'deposit' THEN 1 
    ELSE 0
    END) AS deposit_count,
  SUM (
    CASE
    WHEN txn_type = 'purchase' THEN 1
    ELSE 0
    END) AS purchase_count,
  SUM (
    CASE
    WHEN txn_type = 'withdrawal' THEN 1
    ELSE 0
    END) AS withdrawal_count
  FROM customer_transactions
  GROUP BY customer_id, month
  ) 
SELECT month, COUNT(customer_id) as number_customer
FROM customer_activity
WHERE deposit_count > 1
AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY month
ORDER BY month;
````
|month	    |number_customer|
|-----------|---------------|
|2020-01-01	|168            |
|2020-02-01	|181            |
|2020-03-01	|192            |
|2020-04-01	|70             |

**4. What is the closing balance for each customer at the end of the month?**
- Create a CTE to mark customers who make deposit, purchase and withdrawal, sum the total of those, grouped by each customer_id
- With the 3 new columns: deposit_count, purchase_count, withdrawal_count, we can then filter out who make more than 1 deposit and at least 1 purchase or 1 withdrawal each month
````sql
-- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    -- This defines the window frame for the SUM() window function.
    -- ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW means:
    -- UNBOUNDED PRECEDING → Start from the very first row (earliest month) in the partition (i.e., for that customer).
    -- CURRENT ROW → Include the current row in the sum.
    -- This ensures that the sum accumulates from the first transaction up to the current row.
    WITH monthly_transaction AS (
      SELECT customer_id,
      date_trunc('month',txn_date)::date as month,
      SUM (CASE
           WHEN txn_type = 'deposit' THEN txn_amount
           WHEN txn_type IN ('purchase', 'withdraw') THEN -txn_amount
           ELSE 0
           END) AS net_monthly_change
      FROM customer_transactions
      GROUP BY customer_id, month
      ),
      cumulative_balance AS (
        SELECT customer_id,
        month,
        SUM(net_monthly_change) OVER (
          PARTITION BY customer_id
          ORDER BY month
          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
          ) AS closing_balance
        FROM monthly_transaction
        )
    SELECT *
    FROM cumulative_balance
    ORDER BY customer_id, month;
````

| customer_id | month      | closing_balance |
| ----------- | ---------- | --------------- |
| 1           | 2020-01-01 | 312             |
| 1           | 2020-03-01 | -640            |
| 2           | 2020-01-01 | 549             |
| 2           | 2020-03-01 | 610             |
| 3           | 2020-01-01 | 144             |
| 3           | 2020-02-01 | -821            |
| 3           | 2020-03-01 | -821            |
| 3           | 2020-04-01 | -328            |
| 4           | 2020-01-01 | 848             |
| 4           | 2020-03-01 | 655             |
| 5           | 2020-01-01 | 1780            |
| 5           | 2020-03-01 | 389             |
| 5           | 2020-04-01 | 389             |
| 6           | 2020-01-01 | 733             |
| 6           | 2020-02-01 | 117             |
| 6           | 2020-03-01 | 1898            |
| 7           | 2020-01-01 | 964             |
| 7           | 2020-02-01 | 3173            |
| 7           | 2020-03-01 | 2606            |
| 7           | 2020-04-01 | 3221            |
| 8           | 2020-01-01 | 587             |
| 8           | 2020-02-01 | 587             |
| 8           | 2020-03-01 | 1076            |
| 8           | 2020-04-01 | 104             |
| 9           | 2020-01-01 | 849             |
| 9           | 2020-02-01 | 849             |
| 9           | 2020-03-01 | 2225            |
| 9           | 2020-04-01 | 3178            |
| 10          | 2020-01-01 | -138            |
| 10          | 2020-02-01 | 972             |
| 10          | 2020-03-01 | -157            |
| 10          | 2020-04-01 | -1674           |
