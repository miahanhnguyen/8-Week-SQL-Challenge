1. How many pizzas were ordered?
````sql
SELECT count(order_id) as number_pizza
FROM cleaned_customer_orders;
````

2. How many unique customer orders were made?
````sql
SELECT count(distinct order_id) as unique_order
FROM cleaned_customer_orders;
````

3. How many successful orders were delivered by each runner?
````sql
SELECT distinct runner_id, count(order_id) as delivery_number
FROM cleaned_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;
````

4. How many of each type of pizza was delivered?
````sql
SELECT distinct pizza_id, count(pizza_id) as delivered_pizza
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY pizza_id;
````

5. How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT c.customer_id, p.pizza_name
FROM cleaned_customer_orders c
JOIN pizza_names p
ON c.pizza_id = p.pizza_id
ORDER BY c.customer_id;
````sql

6. What was the maximum number of pizzas delivered in a single order?
