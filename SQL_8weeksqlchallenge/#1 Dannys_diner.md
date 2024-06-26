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
         customer_id, COUNT(distinct(order_date)) as Number_of_visits
     FROM
         sales
     GROUP BY customer_id

<img width="142" alt="1" src="https://github.com/neecao/master/assets/85617864/222816e1-4172-4a54-9c8c-325ff66aad72">


### 3. What was the first item from the menu purchased by each customer?
     SELECT customer_id,
             product_name
      FROM   (SELECT s.customer_id,
                     product_name,
                     dense_rank ()
                       OVER(
                         partition BY s.customer_id
                         ORDER BY order_date) AS rn
              FROM   sales s
                     JOIN menu
                       ON menu.product_id = s.product_id) sub
      WHERE  rn = 1 
      GROUP BY customer_id, product_name;

Result with ROW_NUMBER

<img width="128" alt="3" src="https://github.com/neecao/master/assets/85617864/3e970e6d-3bdb-4431-a116-c00a8ca2c3d7">

Result with DENSE_RANK

<img width="131" alt="1" src="https://github.com/neecao/master/assets/85617864/50fd5d67-047b-4f8c-b2fb-4fe1fc1d41c6">


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
      
<img width="187" alt="4" src="https://github.com/neecao/master/assets/85617864/98bfda11-d6d0-4238-b19f-ee17c6ca541c">

### 6. Which item was purchased first by the customer after they became a member?
      WITH tbl
           AS (SELECT s.customer_id,
                      Min(order_date) AS first_order
               FROM   sales s
                      JOIN members m
                        ON m.customer_id = s.customer_id
               WHERE  order_date > join_date
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

<img width="129" alt="5" src="https://github.com/neecao/master/assets/85617864/e7c3dcf4-a5d6-49c5-81b4-e929409d553c">


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

I first create a master table called tbl that joins all the sales records, member’s info and menu. I then used CASE function to add a column named member to classifie if that order is made when that customer is a member (order_date>= join_date = ‘Y’) or not yet a member (Else ‘N’). 

I then create another table called customer_point to calculate points using CASE. If the product_name = sushi, then multiply price by 20. When customer is the member (column member=Y), the product is not sushi (<> sushi), and the order_date is within the first week since the member join_date (join_date + interval 6 day), then price is multiplied by 20. Else other case price is multiplied by 10.

Select customer_id, aggregate function SUM of point from table customer_point, group by customer_id.

      WITH tbl
           AS (SELECT s.customer_id,
                      order_date,
                      s.product_id,
                      product_name,
                      join_date,
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
                             AND ( order_date BETWEEN join_date AND Date_add(join_date, interval 6 day) )
                                 THEN price *20
                                 ELSE price * 10
                      END AS point
               FROM   tbl
               WHERE  Month(order_date) < 2)
      SELECT customer_id,
             SUM(point)
      FROM   customer_point
      GROUP  BY customer_id 

<img width="118" alt="2" src="https://github.com/neecao/master/assets/85617864/b6070cb8-b452-4cfc-b4f6-e1abcdcee417">


### Results/Findings

1. What is the total amount each customer spent at the restaurant?
Customer A spent the most at the restaurant, 76USD, followed by Customer B comes with 74 USD. Customer C comes in third with 36 USD

3. How many days has each customer visited the restaurant?
Customer A visited the restaurants 4 days
Customer B visited the restaurants 6 days
Customer C visited the restaurants 2 days

4. What was the first item from the menu purchased by each customer?
Customer A bought sushi and curry first, customer B bought curry and lastly customer C bought ramen in his first order

6. What is the most purchased item on the menu and how many times was it purchased by all customers?
The most popular item in the menu is ramen, which was bought 8 times

8. Which item was the most popular for each customer?
Customer A’s and C’s most ordered dish is ramen. Meanwhile customer B ordered equally ramen, sushi and curry

9. Which item was purchased first by the customer after they became a member?
After became member, customer A bought ramen while customer B bought sushi
10. Which item was purchased just before the customer became a member?
Before became a member, customer A bought sushi and curry: customer B bought sushi

11. What is the total items and amount spent for each member before they became a member?
Before they become a member, customer A had order 2 times, accounted for 25USD; customer B had order 3 times with the total sum of 40USD

12. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
Customer A has accumulated 860pts, customer B 940pts, and customer C 360pts
13. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
Since their points are doubled on all items during the first week after joining the membership program, customer A has 1370pts, customer B has 9820pts, and customer C receive 360pts at the end of January


### Recommendations
Customer A has accumulated the highest value thus far, totaling $76 over 4 visits. Customer B has made 6 visits, yet their total order value is lower than that of Customer A. 
Danny should conduct a more in-depth analysis of Customer A to create a suitable customer persona and attract more customers with similar characteristics.

Despite ramen being the bestseller, both Customer A and B opted to become members after ordering sushi. Furthermore, Customer B continued to order sushi right after becoming a member, indicating that sushi might be the decisive factor in their decision to join

