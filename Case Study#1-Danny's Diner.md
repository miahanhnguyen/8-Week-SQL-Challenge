# Case Study #1 - Danny's Diner
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/0f83f966-708b-4684-b347-a3a19f220b4b)

**1. What is the total amount each customer spent at the restaurant?**
````sql
SELECT sales.customer_id, sum(menu.price) total_spent
FROM sales
JOIN menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**2. How many days has each customer visited the restaurant?**
````sql
SELECT customer_id, count(distinct order_date) as number_days
FROM sales
GROUP BY customer_id;
````
| customer_id | number_days |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

**3. What was the first item from the menu purchased by each customer?**
````sql
WITH rank_sales as (
  SELECT *,
  row_number () over (partition by customer_id order by order_date) as rank
  FROM sales
  )
SELECT rs.customer_id, rs.order_date, m.product_name
FROM rank_sales rs
JOIN menu m
	ON rs.product_id = m.product_id
 WHERE rank = 1;
````
| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
SELECT m.product_name, count(s.product_id) as total_purchase
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY total_purchase DESC -- From highest to lowest purchase
LIMIT 1; -- LIMIT to the top purchase
````
| product_name | total_purchase |
| ------------ | -------------- |
| ramen        | 8              |

**5. Which item was the most popular for each customer?**
````sql
WITH count_purchase AS (
  SELECT customer_id, product_id, count(product_id) as purchase_count
  FROM sales
  GROUP BY customer_id, product_id
  ),
  popularity as (
  SELECT cp.customer_id, m.product_name,
    rank () over(partition by cp.customer_id order by cp.purchase_count DESC) as rank
  FROM count_purchase cp
  JOIN menu m
	ON m.product_id = cp.product_id
  )
SELECT customer_id, product_name, rank
FROM popularity
WHERE rank = 1; -- WHERE function can only by applied on the column already in the table
````
| customer_id | product_name | rank |
| ----------- | ------------ | ---- |
| A           | ramen        | 1    |
| B           | curry        | 1    |
| B           | sushi        | 1    |
| B           | ramen        | 1    |
| C           | ramen        | 1    |

**6. Which item was purchased first by the customer after they became a member?**
````sql
WITH after_membership AS (
  SELECT s.customer_id, s.order_date, s.product_id, mb.join_date
  FROM sales s
  JOIN members mb
      ON s.customer_id = mb.customer_id
  WHERE s.order_date >= mb.join_date
  ),
  rank_purchase AS(
  SELECT amb.customer_id, amb.product_id, amb.order_date,
  row_number() over(partition by amb.customer_id order by amb.order_date) as rank
  FROM after_membership amb
  )
SELECT rp.customer_id, m.product_name, rp.order_date as purchase_after_membership
FROM rank_purchase rp
JOIN menu m
	ON m.product_id = rp.product_id
WHERE rp.rank = 1;
````
| customer_id | product_name | purchase_after_membership |
| ----------- | ------------ | ------------------------- |
| B           | sushi        | 2021-01-11                |
| A           | curry        | 2021-01-07                |

**7. Which item was purchased just before the customer became a member?**
````sql
WITH before_membership AS (
  SELECT s.customer_id, s.order_date, s.product_id, mb.join_date
  FROM sales s
  JOIN members mb
      ON s.customer_id = mb.customer_id
  WHERE s.order_date < mb.join_date
  ),
  rank_purchase AS(
  SELECT bmb.customer_id, bmb.product_id, bmb.order_date,
  row_number() over(partition by bmb.customer_id order by bmb.order_date DESC) as rank
  FROM before_membership bmb
  )
SELECT rp.customer_id, m.product_name, rp.order_date as purchase_before_membership
FROM rank_purchase rp
JOIN menu m
	ON m.product_id = rp.product_id
WHERE rp.rank = 1;
````
| customer_id | product_name | purchase_before_membership |
| ----------- | ------------ | -------------------------- |
| B           | sushi        | 2021-01-04                 |
| A           | sushi        | 2021-01-01                 |

**8. What is the total items and amount spent for each member before they became a member?**
````sql
WITH before_membership AS (
  SELECT s.customer_id, s.order_date, s.product_id, mb.join_date
  FROM sales s
  JOIN members mb
      ON s.customer_id = mb.customer_id
  WHERE s.order_date < mb.join_date
  )
 SELECT DISTINCT bmb.customer_id, count(bmb.product_id), sum(m.price)
 FROM before_membership bmb
 JOIN menu m
 	ON bmb.product_id = m.product_id
 GROUP BY bmb.customer_id;
````
| customer_id | count | sum |
| ----------- | ----- | --- |
| A           | 2     | 25  |
| B           | 3     | 40  |

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
````sql
WITH after_membership AS (
  SELECT s.customer_id, s.order_date, s.product_id, mb.join_date
  FROM sales s
  JOIN members mb
      ON s.customer_id = mb.customer_id
  WHERE s.order_date >= mb.join_date
  )
 SELECT amb.customer_id, 
 SUM(
 CASE
 	WHEN m.product_name = 'sushi' THEN m.price*10*2
    ELSE m.price*10
 END) AS Points
 FROM after_membership amb
 JOIN menu m
 	ON m.product_id = amb.product_id
 GROUP BY amb.customer_id;
````
| customer_id | points |
| ----------- | ------ |
| B           | 440    |
| A           | 510    |

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
````sql
WITH after_membership AS (
  SELECT s.customer_id, s.order_date, s.product_id, mb.join_date
  FROM sales s
  JOIN members mb
      ON s.customer_id = mb.customer_id
  WHERE s.order_date >= mb.join_date AND s.order_date < '2021-02-01'
  )
SELECT amb.customer_id,
SUM(
  CASE
  -- 2x points if within the first 7 days after membership
  WHEN amb.order_date - amb.join_date < 7 THEN m.price*10*2
  -- 2x points if ordering sushi
  WHEN m.product_name = 'sushi' THEN m.price*10*2
  -- Regular points for other case
  ELSE m.price*10
  END) AS Points
FROM after_membership amb
JOIN menu m
	ON m.product_id = amb.product_id
GROUP BY amb.customer_id;
````
| customer_id | points |
| ----------- | ------ |
| B           | 320    |
| A           | 1020   |

##Schema (PostgreSQL v13)

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');
[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
