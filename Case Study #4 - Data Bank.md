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
