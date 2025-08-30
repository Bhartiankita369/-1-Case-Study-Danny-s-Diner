## Case Study Questions

### 1. What is the total amount each customer spent at the restaurant?

```SQL
SELECT customer_id, SUM (price)
FROM dannys_diner.menu m 
JOIN dannys_diner.sales s 
ON m.product_id = s.product_id 
GROUP BY customer_id
ORDER BY customer_id 
```


|customer_id |price | SUM (m.price)|
|----------- | ---- | -----------  |
|A           |    10|            76|
|B           |    10|            74|
|C           |    12|            36|


### 2. How many days has each customer visited the restaurant?
````sql
Select  customer_id ,count(Distinct order_date) 
from dannys_diner.sales 
Group by customer_id
```

customer_id|COUNT (DISTINCT order_date)|
-----------|---------------------------|
A          |                          4|
B          |                          6|
C          |                          2|

#### Comment: 
It is important here to use **COUNT(DISTINCT ...)** to find the number of days each customer visited the restaurant. If we don't use **DISTINCT** we might end up with a larger number as customers could visit the restaurant more than once. 


### 3.  What was the first item from the menu purchased by each customer?

Select customer_id , product_name
from 
	(Select customer_id, product_name, 
Dense_rank()over(partition by customer_id 
order by order_date) as rn
from dannys_diner.sales s
inner join dannys_diner.menu m
on s.product_id = m.product_id)
where rn = 1 
group by customer_id , product_name


customer_id|product_name|
-----------|------------|
A          |curry       |
A          |sushi       |
B          |curry       |
C          |ramen       |


