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


```sql
Select  customer_id ,count(Distinct order_date) 
from dannys_diner.sales 
Group by customer_id
```

|customer_id|COUNT (DISTINCT order_date)|
|-----------|---------------------------|
|A          |                          4|
|B          |                          6|
|C          |                          2|

#### Comment: 
It is important here to use **COUNT(DISTINCT ...)** to find the number of days each customer visited the restaurant. If we don't use **DISTINCT** we might end up with a larger number as customers could visit the restaurant more than once. 


### 3.  What was the first item from the menu purchased by each customer?

```sql

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
```

|customer_id|product_name|
|-----------|------------|
|A          |curry       |
|A          |sushi       |
B          |curry       |
C          |ramen       |


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
Select count(s.product_id) as most_purchased, m.product_name, m.product_id
from dannys_diner.menu m
inner join dannys_diner.sales s 
on s.product_id = m.product_id
group by m.product_id, m.product_name
```


|most_purchased|product_name|product_id|
|--------------|------------|----------|
|             8|ramen       |         3|
|             4|curry       |         2|
|             3|sushi       |         1|


### 5. Which item was the most popular for each customer?

```sql
with pop_item as (
Select m.product_name, s.customer_id, count(m.product_id)as order_count , 
Dense_rank() over(partition by s.customer_id 
order by count(m.product_id) desc) as rn
from dannys_diner.sales s
join dannys_diner.menu m
on s.product_id = m.product_id
group by s.customer_id, m.product_name)

Select product_name, customer_id, order_count
from pop_item
where rn = 1 
```

product_name|customer_id|order_count|
------------|-----------|-----------|
ramen       |A          |          3|
sushi       |B          |          2|
ramen       |B          |          2|
curry       |B          |          2|
ramen       |C          |          3|



### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH first_item AS (
SELECT s.customer_id, s.order_date, m.join_date, s.product_id,
DENSE_RANK () OVER (PARTITION BY s.customer_id 
ORDER BY s.order_date) AS order_date_rank
FROM dannys_diner.sales s
JOIN dannys_diner.members m 
ON s.customer_id = m.customer_id
WHERE s.order_date >= m.join_date)

SELECT o.customer_id, m2.product_name, o.order_date, o.product_id
FROM dannys_diner.menu m2
JOIN first_item o 
ON o.product_id = m2.product_id
WHERE order_date_rank = 1
order by customer_id
```

customer_id|product_name|order_date|product_id|
-----------|------------|----------|----------|
A          |curry       |2021-01-07|         2|
B          |sushi       |2021-01-11|         1|


### 7. Which menu item(s) was purchased just before the customer became a member and when?

```sql
WITH before_join AS (
SELECT s.customer_id, s.order_date, m.join_date, s.product_id,
DENSE_RANK () OVER(PARTITION BY s.customer_id
ORDER BY s.order_date DESC) AS order_rank
FROM dannys_diner.sales s
JOIN dannys_diner.members m 
ON s.customer_id = m.customer_id
WHERE m.join_date > s.order_date
)
SELECT m2.product_name, b.order_date, b.join_date, b.order_rank, b.customer_id
FROM dannys_diner.menu m2
JOIN before_join b
ON m2.product_id = b.product_id
WHERE order_rank = 1
ORDER BY customer_id
```

|product_name|order_date|join_date |order_rank|customer_id|
|------------|----------|----------|----------|-----------|
|sushi       |2021-01-01|2021-01-07|         1|A          |
|curry       |2021-01-01|2021-01-07|         1|A          |
|sushi       |2021-01-04|2021-01-09|         1|B          |


### 8. What is the total items and amount spent for each member before they became a member?


```sql
Select s.customer_id, Count(s.product_id) as no_product, m2.join_date, sum(m.price) as price_sum,s.order_date
from ((dannys_diner.menu m 
join dannys_diner.sales s 
on s.product_id = m.product_id)
join dannys_diner.members m2 
on s.customer_id = m2.customer_id )
	
where s.order_date < m2.join_date
group by s.customer_id, s.order_date ,m2.join_date
```

customer_id|COUNT (s.product_id)|SUM (m.price)|join_date |order_date|
-----------|--------------------|-------------|----------|----------|
A          |                   2|           25|2021-01-07|2021-01-01|
B          |                   3|           40|2021-01-09|2021-01-04|


### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier how many points would each customer have?

```sql
With food_points as (
	select * , 
	case when product_id = 1 then price*20 
	else price * 10
	end as points
	from dannys_diner.menu)

select s.customer_id, sum(f.points)
from food_points f 
join dannys_diner.sales s
on f.product_id = s.product_id 
group by s. customer_id
order by customer_id
```

|customer_id|SUM(f.points)|
|-----------|-------------|
|A          |          860|
|B          |          940|
|C          |          360|


### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, – not just sushi — how many points do customer A and B have at the end of January?


```sql
 WITH dates AS (
  SELECT
    m.*,
    (m.join_date + INTERVAL '6 days')::date AS join_week,
    -- Last day of Jan 2021:
    (date '2021-02-01' - INTERVAL '1 day')::date AS last_date
  FROM dannys_diner.members m
)
SELECT
  s.customer_id,
  SUM(
    CASE
      WHEN s.product_id = 1 THEN me.price * 20
      WHEN s.order_date BETWEEN d.join_date AND d.join_week THEN me.price * 20
      ELSE me.price * 10
    END
  ) AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu me
  ON me.product_id = s.product_id
JOIN dates d
  ON d.customer_id = s.customer_id
WHERE s.order_date <= d.last_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

customer_id|last_date |order_date|join_week |points|
-----------|----------|----------|----------|------|
A          |2021-01-31|2021-01-01|2021-01-13|  1370|
B          |2021-01-31|2021-01-01|2021-01-15|   820|




