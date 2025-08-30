# 🍜 Case Study #2 - Pizza Runner:  
![image](https://user-images.githubusercontent.com/77920592/199072945-1bdf34ab-bc60-49eb-bcf9-0f21fb9dbe5d.png)

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-2/). 

***

## Business Task
Danny is expanding his new Pizza Empire and at the same time, he wants to Uberize it, so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Data Cleaning: 
### Table: customer_orders
Looking at the customer_orders table below, we can see that there are:

In the exclusions column, there are missing/ blank spaces ' ' and null values.
In the extras column, there are missing/ blank spaces ' ' and null values.
I am cleaning them using a TEMPORARY Table.

````sql
CREATE TEMPORARY TABLE customer_order_temp (
    order_id INT,
    customer_id INT,
    pizza_id INT,
    exclusions VARCHAR(255),
    extras VARCHAR(255),
    order_time DATETIME
) AS
SELECT
    order_id,
    customer_id,
    pizza_id,
    COALESCE(exclusions, '') as exclusions,
    COALESCE(extras, '') as extras,
    order_time
FROM customer_orders;
````

#### Answer:
![image](https://github.com/user-attachments/assets/1a8f699b-b9e9-4d00-80f5-bd7a9b64b719)


### Table: runner_orders

Looking at the table below, we can see that there are:

In the exclusions column, there are missing/ blank spaces ' ' and null values.
In the extras column, there are missing/ blank spaces ' ' and null values
```` sql
CREATE TEMPORARY TABLE runner_orders_temp AS
SELECT
    order_id,
    runner_id,
    COALESCE(pickup_time, ' ') as pickup_time,
    coalesce(trim(case when distance LIKE '%km' THEN replace(distance, 'km', ' ') else distance END),' ') as distance,
    COALESCE(TRIM(REGEXP_REPLACE(duration, '(minutes|minute|min|mins|s)', ' ')),' ') AS duration,
    COALESCE(cancellation, '') as cancellation
FROM runner_orders;
````
#### Answer:
![image](https://github.com/user-attachments/assets/0e486165-9434-4283-af47-2e4d54d79e9b)

## [Question and Solution](#question-and-solution)

***

### A. Pizza Metrics

**1. How many pizzas were ordered?**

````sql
SELECT COUNT(*) AS pizza_order_count
FROM customer_orders_temp;
````
#### Answer:
![image](https://github.com/user-attachments/assets/db127ebb-2a5a-4204-8ace-b7110a9c7254)



**2. How many unique customer orders were made?**
````sql
SELECT count(distinct order_id) as unique_customer_orders
FROM customer_order_temp;
````
#### Answer: 
![image](https://github.com/user-attachments/assets/e47f378f-9df1-4125-8bab-1bc3c08f9d49)


**3. How many successful orders were delivered by each runner?**
````sql
SELECT r.runner_id, count(r.order_id) as successful_order_count
FROM runner_orders_temp as r
where cancellation NOT in ('Customer Cancellation','Restaurant Cancellation')
group by r.runner_id;
````
#### Answer: 
![image](https://github.com/user-attachments/assets/a8c093fa-3444-4206-87ca-ad4ce96cd8de)


**4. How many of each type of pizza was delivered?**
````sql
select p.pizza_id, count(c.order_id) as pizzas_delivered
from pizza_names as p 
join customer_order_temp as c 
on p.pizza_id = c.pizza_id
join runner_orders_temp as r
on c.order_id = r.order_id
where lower(cancellation) not LIKE '%cancellation%'
group by c.pizza_id;
````
#### Answer:
![image](https://github.com/user-attachments/assets/e91a7ba9-5f54-4efe-a891-00c3dd982076)

**5. How many Vegetarian and Meatlovers were ordered by each customer?**
````sql
select c.customer_id, p.pizza_name, 
sum(case 
when p.pizza_name = 'Meatlovers' THEN 1
when p.pizza_name = 'Vegetarian' THEN 1 
END) as order_count
from pizza_names as p 
join customer_order_temp as c 
on p.pizza_id = c.pizza_id
join runner_orders_temp as r
on c.order_id = r.order_id
group by c.customer_id, p.pizza_name
order by c.customer_id;
````
#### Answer:
![image](https://github.com/user-attachments/assets/ebe9a5a9-6ad0-440f-93a2-29f8bd80645d)

**6. What was the maximum number of pizzas delivered in a single order?**
````sql
WITH ranked_ct_pizzas_delivered as (
select c.order_id, count(p.pizza_name) as pizzas_delivered, rank()over(order by count(p.pizza_name) desc) as rnk
from pizza_names as p 
join customer_order_temp as c 
on p.pizza_id = c.pizza_id
join runner_orders_temp as r
on c.order_id = r.order_id
group by c.order_id)

select order_id, max(pizzas_delivered)
from ranked_ct_pizzas_delivered;
````
#### Answer:
![image](https://github.com/user-attachments/assets/e51b9675-726a-4f0f-8e2f-41b5eb424dc0)

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
````sql
select c.customer_id, 
sum(
	case
		when exclusions != '' or extras != '' then 1 else 0 END) as changes,
sum(
	case
		when exclusions = '' and extras = '' then 1 else 0 END) as no_changes        
from pizza_names as p 
join customer_order_temp as c 
on p.pizza_id = c.pizza_id
join runner_orders_temp as r
on c.order_id = r.order_id
where lower(r.cancellation) not LIKE '%cancellation%'
group by c.customer_id;
````
#### Answer:
![image](https://github.com/user-attachments/assets/fdde2b33-1454-4b80-98d6-1fca932bf120)

**8. How many pizzas were delivered that had both exclusions and extras?**
````sql
select  
sum(
	case
		when exclusions != '' and extras != '' then 1 else 0 END) as exclusions_and_extras
from pizza_names as p 
join customer_order_temp as c 
on p.pizza_id = c.pizza_id
join runner_orders_temp as r
on c.order_id = r.order_id
where lower(r.cancellation) not LIKE '%cancellation%'
;
````
#### Answer:
![image](https://github.com/user-attachments/assets/65336a63-028d-42ff-92ea-eb1a88156a25)

**9. What was the total volume of pizzas ordered for each hour of the day?**
````sql
select hour(order_time) as hours, count(*) as total_volume
FROM customer_order_temp 
group by hour(order_time)
order by hours;
````
#### Answer:
![image](https://github.com/user-attachments/assets/71940689-490b-492a-93c9-cad869b5f4e2)

**10. What was the volume of orders for each day of the week?**

````sql
select dayname(order_time) as days, count(*) as total_volume
FROM customer_order_temp 
group by dayname(order_time)
order by 2 desc;
````
#### Answer:
![image](https://github.com/user-attachments/assets/fa80250d-f2fa-4499-ad76-3d5f984e8779)

***
### B. Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)?**

````sql

````
#### Answer:



**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
````sql

````
#### Answer: 



**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
````sql

````
#### Answer: 


**4. What was the average distance travelled for each customer?**
````sql

````
#### Answer:


**5. What was the difference between the longest and shortest delivery times for all orders?**
````sql

````
#### Answer:


### C. Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**

````sql

````
#### Answer:



**2. What was the most commonly added extra?**
````sql

````
#### Answer: 



**3. What was the most common exclusion?**
````sql

````
#### Answer: 


**4. Generate an order item for each record in the customers_orders table in the format of one of the following:
Meat Lovers
Meat Lovers - Exclude Beef
Meat Lovers - Extra Bacon
Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**
````sql

````
#### Answer:


**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**
````sql

````
#### Answer:


**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
````sql

````
#### Answer:


### D. Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

````sql

````
#### Answer:



**2. What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra**
````sql

````
#### Answer: 



**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**
````sql

````
#### Answer: 


**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas**
````sql

````
#### Answer:


**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?"**
````sql

````
#### Answer:



## BONUS QUESTIONS

** **
If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?
** **

````sql

````
#### Answer:


** **
