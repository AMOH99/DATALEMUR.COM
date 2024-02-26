# DATALEMUR.COM
DataLemur SQL Interview Question and Answer -
DIFFICULTY MEDIUM

1.Assume you are given the table below on Uber transactions made by users. Write a query to obtain the third transaction of every user. Output the user id, spend and transaction date.

SELECT user_id, spend , transaction_date
FROM(
SELECT user_id, spend , transaction_date,
row_number() OVER(
PARTITION BY user_id 
ORDER BY transaction_date) AS row_number
FROM transactions) AS trans_total
WHERE row_number = 3;

2.Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group. Round the percentage to 2 decimal places in the output.

SELECT age_bucket,
ROUND(100.0 *
SUM(activities.time_spent) FILTER (WHERE activities.activity_type = 'send')/
 SUM(activities.time_spent), 2) AS send_perc, 
ROUND(100.0 *
SUM(activities.time_spent) FILTER (WHERE activities.activity_type = 'open')/
  SUM(activities.time_spent), 2) AS open_perc
FROM activities
INNER JOIN age_breakdown AS age 
  ON activities.user_id = age.user_id 
WHERE activities.activity_type IN ('send', 'open') 
GROUP BY age_bucket;

3.Given a table of tweet data over a specified time period, calculate the 3-day rolling average of tweets for each user. Output the user ID, tweet date, and rolling averages rounded to 2 decimal places.

SELECT
  user_id,
  tweet_date,
 ROUND(
 AVG(tweet_count) OVER (
    PARTITION BY user_id
    ORDER BY tweet_date
      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS rolling_avg_3d 
FROM tweets;

4.Assume you're given a table containing data on Amazon customers and their spending on products in different category, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

with spending AS (
SELECT category,
product,
sum(spend) as total_spend,
RANK() OVER(
PARTITION BY category ORDER BY SUM(spend) DESC)
FROM product_spend
WHERE EXTRACT(YEAR FROM transaction_date) = 2022
GROUP BY category, product)
SELECT category,
       product,
       total_spend
FROM spending
WHERE rank <= 2

5.Write a query to find the top 5 artists whose songs appear most frequently in the Top 10 of the global_song_rank table. Display the top 5 artist names in ascending order, along with their song appearance ranking.

WITH top_10_cte AS (
  SELECT 
    artists.artist_name,
    DENSE_RANK() OVER (
      ORDER BY COUNT(songs.song_id) DESC) AS artist_rank
  FROM artists
  INNER JOIN songs
    ON artists.artist_id = songs.artist_id
  INNER JOIN global_song_rank AS ranking
    ON songs.song_id = ranking.song_id
  WHERE ranking.rank <= 10
  GROUP BY artists.artist_name
)

SELECT artist_name, artist_rank
FROM top_10_cte
WHERE artist_rank <= 5;

6.A senior analyst is interested to know the activation rate of specified users in the emails table. Write a query to find the activation rate. Round the percentage to 2 decimal places.

with confirmation AS (
SELECT
SUM(CASE
WHEN texts.signup_action = 'Confirmed' THEN 1 ELSE 0
END) as confirm,
COUNT(emails.user_id) as total_users
FROM emails
join texts
on emails.email_id = texts.email_id)
select ROUND((SUM(confirm) / SUM(total_users)),2)AS CONFIRM_RATE
FROM confirmation

7.Write a query that effectively identifies the company ID of such Supercloud customers.

WITH supercloud_customer_cte AS(
SELECT
  customer_contracts.customer_id,
 COUNT(DISTINCT products.product_category) AS unique_count
FROM customer_contracts LEFT JOIN products
  ON customer_contracts.product_id = products.product_id
GROUP BY customer_contracts.customer_id)
SELECT customer_id
FROM supercloud_customer_cte
WHERE unique_count = (
  SELECT COUNT(DISTINCT product_category) 
  FROM products)
ORDER BY customer_id;

8.Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the monthly_cards_issued table for a given card. Order the results starting from the biggest issued amount.

with launch AS(
SELECT card_name,
issued_amount,
concat(issue_year,'-',issue_month,'-','01') AS issue_date,
RANK() OVER(
partition by card_name
ORDER BY concat(issue_year,'-',issue_month,'-','01')) AS first_issue
FROM monthly_cards_issued)
SELECT card_name,
issued_amount
FROM launch
WHERE first_issue = 1
ORDER BY issued_amount DESC;
