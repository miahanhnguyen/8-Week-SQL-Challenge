# Case Study #7 - Balanced Tree Clothing Co.
[Case Study #7 - Balanced Tree Clothing Co.](https://8weeksqlchallenge.com/case-study-7/)

<img src="https://github.com/user-attachments/assets/1ababbcd-bd79-4f6f-aef3-bcfc4ec26fd6" alt="Image" width="500" height="520">

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f9f41b1c-50c3-4961-841c-48c247db2285)

### A. High Level Sales Analysis
**Total quantity sold for all products, total generated revenue before discounts, total discount amount**
````sql
SELECT 
	SUM(qty) as total_qty,
    SUM(price * qty) as total_revenue,
    SUM(price * qty * discount/100) as total_discount
FROM balanced_tree.sales;
````

|total_qty|	total_revenue|	total_discount|
|-------|--------|-------|
|45216|	1289453|	149486|
