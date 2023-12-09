# Sakila DVD Rental Database Analysis

## Overview

This project was undertaken as part of the Udacity Nanodegree program, "Programming for Data Science with Python." The goal was to explore the Sakila movie database using SQL to answer four specific questions and create a comprehensive report with visualizations.

## Project Files

- **queries.sql**: This file contains the SQL queries used to answer the four questions posed in the project.

- **report.pdf**: The PDF file contains the detailed report generated as a result of the SQL analysis. It includes visualizations corresponding to each of the questions.

## Entity-Relationship Diagram (ERD)

![ERD](erd.png)

## Questions Explored

1. **What are the most popular family movies?**
   - This query identifies the most popular family movies by counting the rentals for films in specific categories (Animation, Children, Classics, Comedy, Family, Music) that are considered family friendly.

   ```sql
   -- Query for the most popular family movies
   SELECT f.title AS film_title,
       c.name AS category_name,
       count(f.title) AS rental_count
    FROM category AS c
    JOIN film_category AS fc ON c.category_id = fc.category_id
    JOIN film AS f ON f.film_id = fc.film_id
    JOIN inventory AS i ON f.film_id = i.film_id
    JOIN rental AS r ON i.inventory_id = r.inventory_id
    WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    GROUP BY film_title, category_name
    ORDER BY category_name, film_title;

2. **How did the two DVD stores fare?**
   - This query provides information on the number of rentals for each store, broken down by month and year.

    ```sql
    -- Query for store performance
    SELECT date_part('month', r.rental_date) AS rental_month,
           date_part('year', r.rental_date) AS rental_year,
           c.store_id,
           count(*) AS count_rentals
    FROM customer AS c
    JOIN rental AS r ON c.customer_id = r.customer_id
    GROUP BY rental_month, rental_year, c.store_id
    ORDER BY count_rentals DESC;

3. **Who were the top ten customers and how much did they spend monthly?**
   - This query identifies the top 10 customers based on their total payments and provides information on their monthly spending.
   ```sql
     -- Query for top 10 customers and their monthly spending
      WITH top_customers AS
        (SELECT c.customer_id,
                sum(p.amount) AS total_payment
         FROM customer AS c
         JOIN payment AS p ON c.customer_id = p.customer_id
         GROUP BY c.customer_id
         ORDER BY total_payment DESC
         LIMIT 10)
      SELECT date_trunc('month', p.payment_date) AS payment_month,
             concat(c.first_name, ' ', c.last_name) AS full_name,
             count(p.amount) AS count_monthly_payments,
             sum(p.amount) AS total_monthly_payments
      FROM customer AS c
      JOIN payment AS p ON c.customer_id = p.customer_id
      WHERE c.customer_id IN (SELECT customer_id FROM top_customers)
      GROUP BY payment_month, full_name
      ORDER BY full_name;

4. **Among the top 10 customers, what was their monthly payment difference, and which customer had the highest monthly difference?**
   - This query calculates the monthly payment difference for the top 10 customers and identifies the customer with the highest monthly difference.
  
   ```sql
     -- Query for monthly payment difference among top 10 customers
      WITH top_customers AS
        (SELECT c.customer_id,
                sum(p.amount) AS total_payment
         FROM customer AS c
         JOIN payment AS p ON c.customer_id = p.customer_id
         GROUP BY c.customer_id
         ORDER BY total_payment DESC
         LIMIT 10),
           top_monthly_payments AS
        (SELECT date_trunc('month', p.payment_date) AS payment_month,
                concat(c.first_name, ' ', c.last_name) AS full_name,
                count(p.amount) AS count_monthly_payments,
                sum(p.amount) AS total_monthly_payments
         FROM customer AS c
         JOIN payment AS p ON c.customer_id = p.customer_id
         WHERE c.customer_id IN (SELECT customer_id FROM top_customers)
         GROUP BY payment_month, full_name
         ORDER BY full_name)
      SELECT payment_month,
             full_name,
             count_monthly_payments,
             total_monthly_payments,
             total_monthly_payments - lag(total_monthly_payments) OVER (ORDER BY full_name) AS monthly_difference
      FROM top_monthly_payments;

