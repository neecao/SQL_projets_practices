# Case Study #1 - Danny's Diner

## Project overview
For full information [8weeksqlchallenge](https://8weeksqlchallenge.com/case-study-1/ )

## Tools
mysql

## Key questions & Solutions
### 1. What is the total amount each customer spent at the restaurant?
        SELECT 
            s.customer_id, SUM(price)
        FROM
            sales s
                LEFT JOIN
            menu ON menu.product_id = s.product_id
        GROUP BY s.customer_id


<img width="112" alt="Capture d’écran 2024-04-18 211249" src="https://github.com/neecao/master/assets/85617864/9ee707af-1b34-4141-b390-813f6a501ea7">

### 2. How many days has each customer visited the restaurant?
            SELECT 
             customer_id, COUNT(order_date) as Number_of_visits
         FROM
             sales
         GROUP BY customer_id

<img width="139" alt="Capture d’écran 2024-04-19 084118" src="https://github.com/neecao/master/assets/85617864/002bbf31-c76c-4c42-816e-aa8ac1d5de3b">


### 3. What was the first item from the menu purchased by each customer?
      SELECT customer_id,
             product_name
      FROM   (SELECT s.customer_id,
                     product_name,
                     Row_number ()
                       OVER(
                         partition BY s.customer_id
                         ORDER BY order_date) AS rn
              FROM   sales s
                     JOIN menu
                       ON menu.product_id = s.product_id) sub
      WHERE  rn = 1 

<img width="130" alt="1" src="https://github.com/neecao/master/assets/85617864/a3c6e998-ad3d-4840-8d98-e1518f7a0fca">


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
         SELECT 
             product_name, COUNT(*) AS Count_order
         FROM
             sales
                 JOIN
             menu ON menu.product_id = sales.product_id
         GROUP BY product_name
         ORDER BY Count_order DESC
         LIMIT 1

<img width="147" alt="2" src="https://github.com/neecao/master/assets/85617864/c0b105f1-d492-429c-8ef8-75bdd643f42e">

### 5. Which item was the most popular for each customer?
      WITH tbl
           AS (SELECT s.customer_id,
                      product_name,
                      Count(s.product_id),
                      Rank()
                        OVER(
                          partition BY customer_id
                          ORDER BY Count(s.product_id) DESC) ranking
               FROM   sales s
                      JOIN menu
                        ON menu.product_id = s.product_id
               GROUP  BY s.customer_id,
                         product_name)
      SELECT customer_id,
             product_name
      FROM   tbl
      WHERE  ranking = 1 

<img width="130" alt="3" src="https://github.com/neecao/master/assets/85617864/af31ab29-0e9f-414f-a0f0-8c4d5ee12c79">

### 6. Which item was purchased first by the customer after they became a member?
      WITH tbl
           AS (SELECT s.customer_id,
                      Min(order_date) AS first_order
               FROM   sales s
                      JOIN members m
                        ON m.customer_id = s.customer_id
               WHERE  order_date >= join_date
               GROUP  BY s.customer_id)
      SELECT tbl.customer_id,
             product_name
      FROM   tbl
             JOIN sales
               ON tbl.customer_id = sales.customer_id
             JOIN menu
               ON sales.product_id = menu.product_id
      WHERE  tbl.first_order = sales.order_date
      ORDER  BY customer_id 

<img width="136" alt="7" src="https://github.com/neecao/master/assets/85617864/100c0fc6-0834-4e70-bb49-85ec17dd6415">

### 7. Which item was purchased just before the customer became a member?
       WITH tbl
                 AS (SELECT s.customer_id,
                            max(order_date) AS first_order
                     FROM   sales s
                            JOIN members m
                              ON m.customer_id = s.customer_id
                     WHERE  order_date <join_date
                     GROUP  BY s.customer_id)
            SELECT tbl.customer_id,
                   product_name
            FROM   tbl
                   JOIN sales
                     ON tbl.customer_id = sales.customer_id
                   JOIN menu
                     ON sales.product_id = menu.product_id
            WHERE  tbl.first_order = sales.order_date
            ORDER  BY customer_id 
            
<img width="142" alt="7" src="https://github.com/neecao/master/assets/85617864/772f476c-ac27-45e8-bf9d-102b289d2c55">

### 8. What is the total items and amount spent for each member before they became a member?
      SELECT 
          members.customer_id, COUNT(sales.product_id), SUM(price)
      FROM
          members
              LEFT JOIN
          sales ON sales.customer_id = members.customer_id
              JOIN
          menu ON menu.product_id = sales.product_id
      WHERE
          order_date < join_date
      GROUP BY members.customer_id

<img width="217" alt="1" src="https://github.com/neecao/master/assets/85617864/0ce7d5c6-7c3b-4265-89f7-5c5130aaf5dc">


### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
      SELECT sub.customer_id,
             Sum(price),
             Sum(points)
      FROM   (SELECT sales.customer_id,
                     product_name,
                     price,
                     CASE
                       WHEN product_name = 'sushi' THEN price * 20
                       ELSE price * 10
                     END AS Points
              FROM   sales
                     JOIN menu
                       ON sales.product_id = menu.product_id) sub
      GROUP  BY sub.customer_id 

<img width="170" alt="2" src="https://github.com/neecao/master/assets/85617864/e0bf064f-818d-4e2b-839c-929c9b912c00">

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
      WITH tbl
           AS (SELECT s.customer_id,
                      order_date,
                      s.product_id,
                      product_name,
                      price,
                      CASE
                        WHEN join_date IS NOT NULL
                             AND order_date >= join_date THEN 'Y'
                        ELSE 'N'
                      END AS member
               FROM   sales s
                      left join menu
                             ON s.product_id = menu.product_id
                      left join members
                             ON members.customer_id = s.customer_id
               ORDER  BY s.customer_id),
           customer_point
           AS (SELECT tbl.customer_id,
                      order_date,
                      product_name,
                      price,
                      CASE
                        WHEN product_name = 'sushi' THEN price * 20
                        WHEN member = 'Y'
                             AND product_name <> 'sushi'
                             AND ( order_date BETWEEN join_date AND Date_add(join_date, interval 7 day) )
                                 THEN price *20
                                 ELSE price * 10
                      END AS point
               FROM   tbl
                      left join members
                             ON members.customer_id = tbl.customer_id
               WHERE  Month(order_date) < 2)
      SELECT customer_id,
             SUM(point)
      FROM   customer_point
      GROUP  BY customer_id 

<img width="125" alt="3" src="https://github.com/neecao/master/assets/85617864/88bba976-5edd-41bb-9290-dc4ecc93b367">


### Results/Findings

### Recommendations

