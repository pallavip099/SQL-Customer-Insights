# SQL_Customer_Insights

# 🍜 Case Study - Danny's Diner
<img width="512" height="512" alt="Image" src="https://github.com/user-attachments/assets/e2732fce-abed-42d4-ba58-809c2bce5206" />

## 📕 Table of Contents
- Business Task
- Case Study Questions
- Bonus Question
- My Solution

---

## 🛠️ Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

---

## ❓ Case Study Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. Join All The Things - Create a table that has these columns: customer_id, order_date, product_name, price, member (Y/N).

---

## 📃 Bonus Question
- Rank All The Things - Based on the table above, add one column: ranking.

---

## 🚀 My Solution

### Q1. What is the total amount each customer spent at the restaurant?
```sql
SELECT c.customer_id,
SUM(p.price) AS total_sales
FROM dannys_diner.sales c
JOIN dannys_diner.menu p
ON c.product_id = p.product_id
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
### OUTPUT
<img width="233" height="126" alt="Image" src="https://github.com/user-attachments/assets/c603562f-48d0-40d6-b663-151f108fcaa7" />


### Q2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id,
COUNT(DISTINCT order_date) AS count_order_date
FROM dannys_diner.sales
GROUP BY customer_id;
```
### OUTPUT
<img width="276" height="120" alt="Image" src="https://github.com/user-attachments/assets/5ee427a9-0728-4de0-b0b3-b04b06c07b33" />


### Q3. What was the first item from the menu purchased by each customer?
```sql
WITH orderRANK AS (
SELECT
customer_id, order_date, product_id,
DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS rnk
FROM dannys_diner.sales 
)

SELECT 
s.customer_id,
s.order_date,
m.product_name
FROM orderRANK s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id 
WHERE s.rnk = 1
GROUP BY s.customer_id, s.order_date, m.product_name;
```
### OUTPUT
<img width="340" height="137" alt="Image" src="https://github.com/user-attachments/assets/289bf891-5d7c-449b-8ad7-fe5b728b3f36" />



### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT
s.product_id,
m.product_name,
COUNT(*) AS most_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.product_id, m.product_name
ORDER BY s.product_id DESC
LIMIT 1;
```
### OUTPUT
<img width="372" height="83" alt="Image" src="https://github.com/user-attachments/assets/650cb228-0699-4f85-b91c-50dfec623cec" />


### Q5. Which item was the most popular for each customer?
```sql
WITH freqRANK AS (
SELECT
  s.customer_id,
  s.product_id,
  COUNT(*) AS purch_freq,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rnk
FROM dannys_diner.sales s
GROUP BY s.customer_id, s.product_id
)

SELECT 
s.customer_id,
m.product_name,
s.purch_freq
FROM freqRANK s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
WHERE s.rnk = 1
ORDER BY s.customer_id;
```

### OUTPUT
<img width="356" height="161" alt="Image" src="https://github.com/user-attachments/assets/b6f20e1b-41e3-4ec3-94f1-f9e824e7db61" />


### Q6. Which item was purchased first by the customer after they became a member?
```sql
WITH overAfterMember AS (
  SELECT
    DISTINCT (s.customer_id),
    mn.product_name,
    s.order_date,
    m.join_date,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rnk
FROM dannys_diner.sales s
JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
JOIN dannys_diner.menu mn
ON s.product_id = mn.product_id
WHERE order_date >= join_date
)

SELECT 
customer_id,
product_name,
order_date,
join_date
FROM overAfterMember
WHERE rnk = 1;
```
### OUTPUT
<img width="437" height="102" alt="Image" src="https://github.com/user-attachments/assets/9859ad23-14cb-42b7-af66-35807b9fea43" />


### Q7. Which item was purchased just before the customer became a member?
```sql
WITH overBeforeMember AS (
 SELECT
   DISTINCT(s.customer_id),
   mn.product_name,
   s.order_date,
   m.join_date,
DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rnk
FROM dannys_diner.sales s
JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
JOIN dannys_diner.menu mn
ON s.product_id = mn.product_id
WHERE s.order_date < m.join_date
)

SELECT 
customer_id,
product_name,
order_date,
join_date
FROM overBeforeMember
WHERE rnk = 1;
```
### OUTPUT
<img width="430" height="123" alt="Image" src="https://github.com/user-attachments/assets/5278a568-59e6-4b0d-a855-f3683c4f5fe9" />


### Q8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
    s.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(mn.price) AS total_price
FROM dannys_diner.sales s
JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
JOIN dannys_diner.menu mn
ON s.product_id = mn.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id
ORDER BY total_price ASC;
```
### OUTPUT
















