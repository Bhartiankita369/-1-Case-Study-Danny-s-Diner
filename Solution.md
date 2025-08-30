## Case Study Questions

### 1. What is the total amount each customer spent at the restaurant?


SELECT customer_id, SUM (price)
FROM dannys_diner.menu m 
JOIN dannys_diner.sales s 
ON m.product_id = s.product_id 
GROUP BY customer_id
ORDER BY customer_id 
