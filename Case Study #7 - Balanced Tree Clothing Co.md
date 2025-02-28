# Case Study #7 - Balanced Tree Clothing Co.
[Case Study #7 - Balanced Tree Clothing Co.](https://8weeksqlchallenge.com/case-study-7/)

<img src="https://github.com/user-attachments/assets/1ababbcd-bd79-4f6f-aef3-bcfc4ec26fd6" alt="Image" width="500" height="520">

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f9f41b1c-50c3-4961-841c-48c247db2285)

### A. High Level Sales Analysis
**1. What was the total quantity sold for all products?**
````sql
SELECT pd.product_name, SUM(s.qty) as total_qty
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s
ON pd.product_id = s.prod_id
GROUP BY pd.product_name;
````
|product_name |	total_qty|
|------------|----------|
|White Tee Shirt - Mens|	3800|
|Navy Solid Socks - Mens|	3792|
|Grey Fashion Jacket - Womens|	3876|
|Navy Oversized Jeans - Womens|	3856|
|Pink Fluro Polkadot Socks - Mens|	3770|
|Khaki Suit Jacket - Womens|	3752|
|Black Straight Jeans - Womens|	3786|
|White Striped Socks - Mens|	3655|
|Blue Polo Shirt - Mens|	3819|
|Indigo Rain Jacket - Womens|	3757|
|Cream Relaxed Jeans - Womens|	3707|
|Teal Button Up Shirt - Mens|	3646|
