# Case Study #2 - Pizza Runner

## Project overview
Prease refer to [8weeksqlchallenge](https://8weeksqlchallenge.com/case-study-2/) for full information 

## Tools
mysql

## Table of content
- [The Dataset Schema](#schema)
- [Data cleaning process](https://github.com/neecao/master/new/master/SQL_8weeksqlchallenge#data-cleaning-process)
- [Case Study Questions + Solutions](#question)
- [Bonus Questions + Solutions](#bonus)
- [Results/Findings](#result)
- [Recommendations](#recommendations)

<div id='schema'/>
  
### The Dataset Schema

<img width="491" alt="Capture d'écran 2024-04-24 214927" src="https://github.com/neecao/master/assets/85617864/475e24f5-95de-4041-a171-d3acc795741c">

<div id='clean'/>
  
### Data cleaning process
  
Treating null values and formating data types in the customer_orders and runner_orders tables
##### Original runner_orders table
<img width="382" alt="Capture d'écran 2024-04-25 213617" src="https://github.com/neecao/master/assets/85617864/79c57147-a352-4888-b793-6a528568148b">

##### Steps taken: 
- Format column distance and duration to contain only numerical values
- Format null value or 'null' string to be blank

             CREATE temporary TABLE runner_orders_temp
      SELECT order_id,
             runner_id,
             CASE
               WHEN pickup_time = 'null' THEN ''
               ELSE pickup_time
             end AS pickup_time,
             CASE
               WHEN distance = 'null' THEN ''
               ELSE REPLACE(distance, 'km', '')
             end AS distance,
             CASE
               WHEN duration = ''
                     OR duration = 'null' THEN ''
               ELSE Regexp_replace(duration, '[a-z]+', '')
             end AS duration,
             CASE
               WHEN cancellation = ''
                     OR cancellation = 'null'
                     OR cancellation IS NULL THEN ''
               ELSE cancellation
             end AS cancellation
      FROM   runner_orders; 
  
- Update data type for column pickup_time (varchar :arrow_right: datetime), duration (varchar:arrow_right:float), distance (varchar:arrow_right:float)
- Any solution to modify data type returns errors because of null values. I had to change SQL setting temporarily to accept invalide value date.
  
      SET SQL_MODE='ALLOW_INVALID_DATES';
      ALTER TABLE runner_orders_temp
        modify COLUMN distance FLOAT NULL,
        modify COLUMN duration FLOAT NULL,
        modify COLUMN pickup_time DATETIME;
  
##### Cleaned version
<img width="380" alt="11" src="https://github.com/neecao/master/assets/85617864/65be78fe-eafc-4c4a-aeff-66a523343039">

    desc runner_orders_temp
<img width="258" alt="Capture d'écran 2024-04-29 202832" src="https://github.com/neecao/master/assets/85617864/5747b55c-2582-4b7b-ae40-e4441e859d3e">

##### Original customer_orders table
<img width="328" alt="2" src="https://github.com/neecao/master/assets/85617864/73530827-5863-45bf-ae27-98cabd5889e4">

##### Steps taken: 
- Format null value or 'null' string in exclusions and extras to be blank

        CREATE temporary TABLE customer_orders_temp
          SELECT order_id,
                 customer_id,
                 pizza_id,
                 CASE
                    when exclusions ='null' or exclusions =''then null
                    else exclusions
                    end as exclusions,
                    case
                    when extras ='null' or extras = '' then null
                    else extras
                    end as extras,
                  order_time
          FROM   customer_orders;

  ##### Cleaned version
<img width="326" alt="Capture d’écran 2024-04-27 172813" src="https://github.com/neecao/master/assets/85617864/d709559b-3379-4ac8-87f6-08403bff8617">

##### Add space between 'Meatlovers' in table pizza_names
      SET SQL_SAFE_UPDATES = 0;
      UPDATE pizza_names 
      SET 
          pizza_name = REPLACE(pizza_name,
              'Meatlovers',
              'Meat lovers')
      WHERE
          pizza_id = '1'

<div id='question'/>

### Case Study Questions + Solutions
#### A. Pizza Metrics
##### 1. How many pizzas were ordered?
   
          SELECT 
              COUNT(pizza_id) AS total_pizza
          FROM
              customer_orders_temp
   
<p align="center"> <img width="120" alt="Capture d'écran 2024-04-29 204215" src="https://github.com/neecao/master/assets/85617864/4f8ae80c-18c8-4322-a480-d2dcade2b460">

##### 2. How many unique customer orders were made?
        SELECT 
            COUNT(DISTINCT order_id) AS count_unique_order
        FROM
            customer_orders_temp

<p align="center"> <img width="120" alt="Capture d'écran 2024-05-01 171350" src = "https://github.com/neecao/master/assets/85617864/ebc63641-da9f-44aa-9eb5-04b7c5fdeab0">

##### 3. How many successful orders were delivered by each runner?
        SELECT 
            runner_id, COUNT(order_id) AS count_order
        FROM
            runner_orders_temp
        GROUP BY runner_id
        ORDER BY runner_id
        
<p align="center"> <img width="120" alt="Capture d'écran 2024-05-01 171829" src ="https://github.com/neecao/master/assets/85617864/ddbf277e-e8e4-4661-a5b5-8eddb37d2cf2">

##### 4. How many of each type of pizza was delivered?
        SELECT 
            pizza_name, COUNT(order_id) AS count
        FROM
            customer_orders_temp co
                JOIN
            pizza_names n ON n.pizza_id = co.pizza_id
        GROUP BY pizza_name

<p align="center"> <img width="120" alt="Capture d'écran 2024-05-01 175318" src="https://github.com/neecao/master/assets/85617864/a6b5b6e1-1c67-4b7b-8206-b071e1a5fda5">

##### 5. How many Vegetarian and Meatlovers were ordered by each customer?
          SELECT 
              customer_id, pizza_name, COUNT(order_id) AS count
          FROM
              customer_orders_temp co
                  JOIN
              pizza_names n ON n.pizza_id = co.pizza_id
          GROUP BY pizza_name , customer_id
          ORDER BY customer_id
<p align="center"> <img width="200" alt="Capture d'écran 2024-05-01 180631" src = "https://github.com/neecao/master/assets/85617864/80a56ca1-7bbd-4f6a-95a6-9c81ffad32f0">

##### 6. What was the maximum number of pizzas delivered in a single order?
        SELECT order_id,
               count
        FROM   (SELECT order_id,
                       Count(pizza_id) AS count,
                       Rank()
                         OVER (
                           ORDER BY Count(pizza_id) DESC ) AS rank_order
                FROM   customer_orders_temp
                GROUP  BY order_id) tbl
        WHERE  rank_order = 1 

<p align="center"> <img width="120" alt="Capture d'écran 2024-05-01 200041" src = "https://github.com/neecao/master/assets/85617864/28833a5c-7372-4f80-b9fd-69ff4cc7f825">

##### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
              WITH change_tracker
                   AS (SELECT customer_id,
                              co.order_id,
                              CASE
                                WHEN exclusions IS NOT NULL
                                      OR extras IS NOT NULL THEN 1
                                ELSE 0
                              END AS change_count,
                              CASE
                                WHEN exclusions IS NULL
                                     AND extras IS NULL THEN 1
                                ELSE 0
                              END AS no_change_count
                       FROM   customer_orders_temp co
                              JOIN runner_orders_temp ro
                                ON co.order_id = ro.order_id
                       WHERE  duration != 0)
              SELECT customer_id,
                     Sum(change_count)    AS pizza_change,
                     Sum(no_change_count) AS pizza_no_change
              FROM   change_tracker
              GROUP  BY customer_id 
<p align="center"> <img width="250" alt="Capture d'écran 2024-05-01 215702" src="https://github.com/neecao/master/assets/85617864/a4d6cc8b-919b-46eb-8e7b-31e72f96cdff">

##### 8.  How many pizzas were delivered that had both exclusions and extras?
              SELECT Count(co.order_id) AS extras_exclusions
              FROM   customer_orders_temp co
                     JOIN runner_orders_temp ro
                       ON co.order_id = ro.order_id
              WHERE  duration != 0
                     AND exclusions IS NOT NULL
                     AND extras IS NOT NULL 
                     
<p align="center"> <img width="120" alt="Capture d'écran 2024-05-01 220724" src="https://github.com/neecao/master/assets/85617864/1ac879d6-a462-4e8d-9a33-2f30d8815d20">

##### 9. What was the total volume of pizzas ordered for each hour of the day?
        SELECT Hour(order_time) AS hour_of_day,
               Count(order_id)  AS count_order
        FROM   customer_orders_temp
        GROUP  BY hour_of_day
        ORDER  BY hour_of_day 

<p align="center"> <img width="200" alt="Capture d'écran 2024-05-01 222646" src="https://github.com/neecao/master/assets/85617864/2e18d381-e047-4977-82bf-50b848970d10">
      
#### 10. What was the volume of orders for each day of the week?

          SELECT Dayname(order_time) AS day_of_week,
                 Count(order_id)     AS count_order
          FROM   customer_orders_temp
          GROUP  BY day_of_week
          ORDER  BY count_order desc 
          
<p align="center">  <img width="200" alt="Capture d'écran 2024-05-01 223931" src="https://github.com/neecao/master/assets/85617864/6192dd0c-106c-46f8-a030-bf1cb93e3bff">

#### B. Runner and Customer Experience
##### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

        SELECT 
            COUNT(*),
            WEEK(registration_date, '2021-01-01') AS week_number
        FROM
            runners
        GROUP BY WEEK(registration_date, '2021-01-01')
        
<img width="150" alt="Capture d'écran 2024-05-02 195442" src="https://github.com/neecao/SQL_projets_practices/assets/85617864/58dcec1e-5ae2-439d-a1b0-f0568b103642">

##### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
      
      SELECT 
          runner_id, AVG(duration) as average_arrival
      FROM
          (SELECT DISTINCT
              runner_id,
                  TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS duration
          FROM
              customer_orders_temp c
          JOIN runner_orders_temp r ON r.order_id = c.order_id
          WHERE
              pickup_time > 0) tbl
      GROUP BY runner_id

<img width="150" alt="Capture d'écran 2024-05-02 212339" src="https://github.com/neecao/SQL_projets_practices/assets/85617864/1e43292c-2665-4066-921c-5c20912b9c4d">
      
##### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

        SELECT 
            num_pizza, AVG(duration)
        FROM
            (SELECT DISTINCT
                c.order_id,
                    COUNT(c.pizza_id) AS num_pizza,
                    TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS duration
            FROM
                customer_orders_temp c
            JOIN runner_orders_temp r ON r.order_id = c.order_id
            WHERE
                pickup_time > 0
            GROUP BY c.order_id) tbl
        GROUP BY num_pizza
        
<img width="150" alt="Capture d'écran 2024-05-02 220107" src="https://github.com/neecao/SQL_projets_practices/assets/85617864/97ca9f92-96cd-4816-880b-425be1116d01">

        
##### 4. What was the average distance travelled for each customer?

        SELECT 
            customer_id, ROUND(AVG(distance), 2)
        FROM
            customer_orders_temp c
                JOIN
            runner_orders_temp r ON c.order_id = r.order_id
        WHERE
            distance != 0
        GROUP BY customer_id
        
<img width="142" alt="Capture d'écran 2024-05-02 221657" src="https://github.com/neecao/SQL_projets_practices/assets/85617864/2033a82e-df22-40dd-8544-7fb2f74bf7a3">

##### 5. What was the difference between the longest and shortest delivery times for all orders?

        SELECT 
            MAX(duration) - MIN(duration) AS difference
        FROM
            runner_orders_temp
        WHERE
            duration != 0

<img width="62" alt="Capture d’écran 2024-05-03 094655" src="https://github.com/neecao/SQL_projets_practices/assets/85617864/f70ed300-fbfe-4949-bcb5-e55eb2b0b8a5">

##### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

        SELECT runner_id
        	,order_id
        	,round(distance / (duration / 60), 2) AS speed_kmh
        	,round(avg(distance / (duration / 60)) OVER (PARTITION BY runner_id), 2) AS runner_avgspeed
        FROM runner_orders_temp
        WHERE duration != 0
        ORDER BY runner_id
        
![Capture d'écran 2024-05-03 100140](https://github.com/neecao/SQL_projets_practices/assets/85617864/7882a6fb-e4ce-41b9-8b46-c1abc3441b0a)

##### 7. What is the successful delivery percentage for each runner?

        SELECT 
            runner_id,
            ROUND((success_order / all_order) * 100) AS success_rate
        FROM
            (SELECT 
                runner_id,
                    SUM(CASE
                        WHEN pickup_time != '' THEN 1
                        ELSE 0
                    END) AS success_order,
                    COUNT(order_id) AS all_order
            FROM
                runner_orders
            GROUP BY runner_id) tbl

![Capture d'écran 2024-05-03 101721](https://github.com/neecao/SQL_projets_practices/assets/85617864/9c4813b5-6c38-4649-810e-50e13234720c)

<div id='bonus'/>

### Bonus Questions + Solutions

<div id='result'/>

### Results/Findings

<div id='recommendations'/>

### Recommendations

