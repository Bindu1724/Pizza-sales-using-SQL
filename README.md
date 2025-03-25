# Pizza sales project using SQL

You need to download the pptx file to see my project. It is not directly available as it is a big file.

## Overview

This project involves a comprehensive analysis of Pizza Sales data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Schema
```sql
create database pizzahut;
create table orders (
order_id int not null,
order_date date not null,
order_time time not null,
primary key(order_id) );

create table orders_details (
order_details_id int not null,
order_id int not null,
pizza_id text not null,
quantity int not null,
primary key(order_details_id) );
```
## Business Problems and Solutions

### 1. Retrieve the total number of orders placed.
```sql
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
```
### 2. Calculate the total revenue generated from pizza sales.
```sql
SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS total_sales
FROM
    orders_details
        JOIN
    pizzas ON pizzas.pizza_id = orders_details.pizza_id;
```
### 3. Identify the highest-priced pizza.
```sql
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```
### 4. Identify the most common pizza size ordered.
```sql
SELECT 
    pizzas.size,
    COUNT(orders_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC;
```
### 5. List the top 5 most ordered pizza types along with their quantities.
```sql
SELECT 
    pizza_types.name, SUM(orders_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```

### 6. Join the necessary tables to find the total quantity of each pizza category ordered.
```sql
select pizza_types.category, sum(orders_details.quantity) as quantity
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id 
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by pizza_types.category
order by quantity desc;
```

### 7. Determine the distribution of orders by hour of the day.
```sql
SELECT 
    HOUR(order_time), COUNT(order_id)
FROM
    orders
GROUP BY HOUR(order_time);
```

### 8. Join relevant tables to find the category-wise distribution of pizzas.
```sql
select category, count(name) from pizza_types
group by category;
```

### 9.  Group the orders by date and calculate the average number of pizzas ordered per day.
```sql
select round(avg(quantity)) as avg_pizzas_ordered_perday from 
(select orders.order_date, sum(orders_details.quantity) as quantity
from orders join orders_details
on orders.order_id = orders_details.order_id
group by orders.order_date) as order_quantity ;
```

### 10.  Determine the top 3 most ordered pizza types based on revenue.
```sql
select pizza_types.name,
sum(orders_details.quantity*pizzas.price) as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by pizza_types.name order by revenue desc limit 3;
```

### 11. Calculate the percentage contribution of each pizza type to total revenue.
```sql
select pizza_types.category,
round(sum(orders_details.quantity*pizzas.price) / (select 
round(sum(orders_details.quantity*pizzas.price), 2) as total_sales
from orders_details join pizzas
on pizzas.pizza_id=orders_details.pizza_id) * 100) as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by pizza_types.category order by revenue desc;
```

### 12.  Analyze the cumulative revenue generated over time.
```sql
select order_date, round(sum(revenue) over(order by order_date))  as cum_revenue
from (select orders.order_date,
sum(orders_details.quantity * pizzas.price) as revenue
from orders_details join pizzas
on orders_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = orders_details.order_id
group by orders.order_date) as sales;
```

### 13.  Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```sql
select category, name, revenue 
from
(select category, name, revenue, 
rank() over(partition by category order by  revenue desc) as rn
from
(select pizza_types.category, pizza_types.name,
sum(orders_details.quantity * pizzas.price) as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by pizza_types.category, pizza_types.name) as a) as b
where rn <=3;
```
