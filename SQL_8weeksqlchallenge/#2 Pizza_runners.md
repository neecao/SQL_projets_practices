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
- Format empty cell to have null value

      CREATE temporary TABLE runner_orders_temp
      SELECT order_id,
             runner_id,
             CASE
               WHEN pickup_time = 'null' THEN NULL
               ELSE pickup_time
             end AS pickup_time,
             CASE
               WHEN distance = 'null' THEN NULL
               ELSE REPLACE(distance, 'km', '')
             end AS distance,
             CASE
               WHEN duration = '' THEN NULL
               WHEN duration = 'null' THEN NULL
               ELSE Regexp_replace(duration, '[a-z]+', '')
             end AS duration,
             CASE
               WHEN cancellation = '' THEN NULL
               WHEN cancellation = 'null' THEN NULL
               ELSE cancellation
             end AS cancellation
      FROM   runner_orders;  
##### Cleaned version
<img width="376" alt="Capture d’écran 2024-04-25 213041" src="https://github.com/neecao/master/assets/85617864/e92814c2-3e71-4c5b-aa2d-d0f1abab317b">

##### Original customer_orders table
<img width="328" alt="2" src="https://github.com/neecao/master/assets/85617864/73530827-5863-45bf-ae27-98cabd5889e4">

##### Steps taken: 
- Format empty cell in exclusions and extras to have null value

        CREATE temporary TABLE customer_orders_temp
          SELECT order_id,
                 customer_id,
                 pizza_id,
                 CASE
                   WHEN exclusions = 'null' THEN NULL
                   WHEN exclusions = '' THEN NULL
                   ELSE exclusions
                 end AS exclusions,
                 CASE
                   WHEN extras = 'null' THEN NULL
                   WHEN extras = '' THEN NULL
                   ELSE extras
                 end AS extras
          FROM   customer_orders;
  
  ##### Cleaned version
<img width="233" alt="1" src="https://github.com/neecao/master/assets/85617864/d2bf5185-a05b-430a-9664-ea5130798b0f">

  
<div id='question'/>

### Case Study Questions + Solutions

<div id='bonus'/>

### Bonus Questions + Solutions

<div id='result'/>

### Results/Findings

<div id='recommendations'/>

### Recommendations

