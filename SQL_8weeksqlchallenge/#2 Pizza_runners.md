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
  
- Update data type for column pickup_time (varchar -> datetime), duration (varchar->float), distance (varchar->float)
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


<div id='question'/>

### Case Study Questions + Solutions

<div id='bonus'/>

### Bonus Questions + Solutions

<div id='result'/>

### Results/Findings

<div id='recommendations'/>

### Recommendations

