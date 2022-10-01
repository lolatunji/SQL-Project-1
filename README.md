# SQL-Project-1


# Project Name: Danny's Dinner

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:


# Data Source: https://8weeksqlchallenge.com/case-study-1/

# Case study questions and queries;

1. What is the total amount each customer spent at the restaurant?

SELECT DISTINCT customer_id, SUM(price) AS total_amount_spent

FROM dannys_diner.sales

JOIN dannys_diner.menu

USING(product_id)

GROUP BY customer_id

ORDER BY SUM(price) DESC;

![image](https://user-images.githubusercontent.com/108012164/193408431-38c64cc5-d432-4d78-b8dc-ca0f8ac5c23d.png)

2. How many days has each customer visited the restaurant? 

SELECT customer_id, COUNT (DISTINCT order_date) AS number_days

FROM dannys_diner.sales

GROUP BY customer_id

ORDER BY customer_id;

![image](https://user-images.githubusercontent.com/108012164/193408589-451c4bec-1f1d-4d14-8736-b28a57fbcd20.png)

3. What was the first item from the menu purchased by each customer?

SELECT DISTINCT order_date, customer_id, product_name AS first_order

FROM

(SELECT DISTINCT order_date, customer_id, product_name, ROW_NUMBER () OVER(PARTITION BY customer_id ORDER BY order_date) AS first_item_ordered

 FROM dannys_diner.menu
 
 JOIN dannys_diner.sales
 
 USING (product_id)) AS f
 
 WHERE f.first_item_ordered = 1;
 
 ![image](https://user-images.githubusercontent.com/108012164/193408656-e4a625cf-0f15-41c1-acdd-4a2ca1936dae.png)
 
 4. What is the most purchased item on the menu and how many times was it purchased by all customers? 
 
SELECT product_name, COUNT(product_id) AS most_purchased_item

FROM dannys_diner.sales

JOIN dannys_diner.menu

USING (product_id)

GROUP BY product_name

ORDER BY  COUNT(product_id) DESC;

![image](https://user-images.githubusercontent.com/108012164/193408772-efaa6b92-22a0-4c4b-bb3b-42adc7eb7ce0.png)

5. Which item was the most popular for each customer?

WITH CTE AS (

     SELECT s.customer_id, m.product_name, COUNT(s.product_id),
     
     DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id) DESC) AS most_popular
     
     FROM dannys_diner.menu AS m
 
     JOIN dannys_diner.sales AS s
     
     USING(product_id) AS ms
     
	 GROUP BY s.customer_id, m.product_name)
   
	 SELECT customer_id, product_name
   
	 FROM CTE
   
	 WHERE  most_popular = 1;
   
   ![image](https://user-images.githubusercontent.com/108012164/193408843-ddc40c70-f910-4582-bc94-c0c07340902a.png)
   
   6. Which item was purchased first by the customer after they became a member?
   
WITH member_cte AS(

           SELECT customer_id, product_name, order_date, join_date,
           
           ROW_NUMBER () OVER(PARTITION BY customer_id) AS first_item_m
           
           FROM dannys_diner.sales
           
           JOIN dannys_diner.members
           
           USING(customer_id)
           
           JOIN dannys_diner.menu
           
           USING(product_id)
           
	       WHERE order_date >= join_date)
         
	       SELECT customer_id, product_name
         
		   FROM member_cte
       
		   WHERE first_item_m =1;
       
![image](https://user-images.githubusercontent.com/108012164/193408918-5ff39aef-45e4-471f-ad11-351bcb3ce11a.png)

7. Which item was purchased just before the customer became a member?

WITH purc_before_member_cte AS (

       SELECT customer_id, product_name, order_date, join_date,
       
       ROW_NUMBER () OVER(PARTITION BY customer_id) AS purc_before_member
       
       FROM dannys_diner.members
       
       JOIN dannys_diner.sales 
       
       USING (customer_id)
       
       JOIN dannys_diner.menu
       
       USING(product_id)
       
       WHERE order_date < join_date)
       
	   SELECT customer_id, product_name
     
	   FROM purc_before_member_cte
     
	   WHERE purc_before_member = 1; 

![image](https://user-images.githubusercontent.com/108012164/193409034-36877f7e-1b9c-42e3-91a6-4e3dc901dc9d.png)

8. What is the total items and amount spent for each member before they became a member?

SELECT customer_id, SUM(price) AS amount_before_member, COUNT(product_id)

FROM dannys_diner.members

JOIN dannys_diner.sales

USING (customer_id)

JOIN dannys_diner.menu

USING(product_id)

WHERE order_date < join_date

GROUP BY customer_id

ORDER BY customer_id;

![image](https://user-images.githubusercontent.com/108012164/193409118-5f2b5a16-ac7d-4e05-90e1-72ce1631ab18.png)

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how manypoints would each customer have?

SELECT customer_id, SUM(point)

FROM

    (SELECT customer_id, product_name,
    
    CASE WHEN product_name = 'sushi' THEN 20* price ELSE 10* price
    
    END AS point
    
    FROM dannys_diner.menu
    
    JOIN dannys_diner.sales 
    
    USING (product_id)) y
    
	GROUP BY customer_id
  
	ORDER BY customer_id;
  
  ![image](https://user-images.githubusercontent.com/108012164/193409207-3a805e92-849c-457d-8ca1-bad9d350d2b6.png)
  
 10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

SELECT customer_id,

	sum(mem_point) AS points_earn_in_JAN
FROM 

	(
	SELECT
  
		s.customer_id,
    
		order_date,
    
		product_name,
    
		price,
    
		join_date,
    
		CASE
    
			WHEN order_date BETWEEN join_date AND join_date + INTERVAL '7days'
      
			OR product_name = 'sushi' THEN 20 * price
      
			ELSE 10 * price
      
		END AS mem_point
    
	FROM
  
		dannys_diner.sales s
    
	JOIN dannys_diner.menu m ON
  
		s.product_id = m.product_id
    
	JOIN dannys_diner.members mb ON
  
		s.customer_id = mb.customer_id
    
	ORDER BY
  
		s.customer_id,
    
		order_date)sub
    
WHERE EXTRACT(MONTH
  
FROM order_date)= 1
  
GROUP BY customer_id;

![image](https://user-images.githubusercontent.com/108012164/193409298-a5154d89-2fa3-4ff6-b3fa-1da51830a40e.png)

# Findings and Recommendations

1. Ramen is the best selling item on the menus list.

2. Customer A and B spent much in danny's dinner restaurant.

3. Customer B purchase all items and order them equally, while Customer A and C purchase Ramen the most.









