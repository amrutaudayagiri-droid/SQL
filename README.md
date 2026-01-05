# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

- Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question & Solution
Please join me in executing the queries using MySQL on https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138. It would be great to work together on the questions!

**1.What is the total amount each customer spent at the restaurant?

````sql
select customer_id,sum(price) as Total_Amount_Spent
from sales as s inner join menu as m using(product_id)
 group by customer_id;
````
####  
#### 
***

**2.How many days has each customer visited the restaurant?

````sql
select customer_id,count(distinct order_date) as No_of_visit_Days 
from sales group by customer_id;
````
####  
####
***

**3.What was the first item from the menu purchased by each customer?

````sql
select * from(
select *,row_number() over(partition by customer_id order by order_date) as rn 
 from sales as s inner join menu as m 
using(product_id)) as T  where rn = 1;
````
####  
***

**4.What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
select product_name,count(*) as cnt from sales as s
inner join menu as m using(product_id) group by product_name
order by cnt desc limit 1;
````
####  
***

**5.Which item was the most popular for each customer?
````sql
select * from (
select customer_id,product_name,count(*) as cnt,
rank() over(partition by customer_id order by count(*) desc) as rn
from sales as s inner join menu as m using(product_id)
group by customer_id,product_name ) as t where rn = 1;
````
####  
***

**6.Which item was purchased first by the customer after they became a member?
````sql
with CTE1 as (
select s.*,m.price,m.product_name,mb.join_date  from sales as s inner join menu as m using (product_id)
inner join members as mb on s.customer_id = mb.customer_id and order_date > join_date),CTE2 as (

select *,rank() over(partition by customer_id order by order_date asc) as rn from CTE1)
select  customer_id, product_name from CTE2 where rn = 1;
````
####  
***

**7.Which item was purchased just before the customer became a member?
````sql
select customer_id,count(order_date) as num_of_orders from (
select *from sales as s inner join members as m using(customer_id) where order_date > join_date)
as t group by customer_id;
````
####  
***

**8.What is the total items and amount spent for each member before they became a member?
````sql
with CTE1 as (
select s.customer_id,count(*) as total_items, sum(price) as total_amount_spent from sales as s
inner join menu as m using(product_id)
inner join members as mb on s.customer_id = mb.customer_id and s.order_date < mb.join_date
group by s.customer_id order by customer_id )
select *from CTE1 
````
####  
***

**9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
select customer_id,sum(case when product_name  ='Sushi' then price*10*2 else price*10 end) as total_points
from sales as s inner join menu as m using(product_id) group by customer_id;
````
####  
***

**10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
select customer_id,sum(case when s.order_date between join_date and
date_add(join_date,Interval 6 day) then price * 20 when product_name = 'Sushi' then price * 20 
else price * 10 end)  as total_points
from sales as s inner join menu as m using(product_id) inner join members as mb using(customer_id)
where order_date <= "2021-01-31"
group by customer_id;

````
####  
***







