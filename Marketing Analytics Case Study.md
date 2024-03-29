# 🎬 Marketing Analytics Case Study

## 📌 1.0 Overview

### Business Task

We (Leadership team) have been asked to support the Customer Analytics team at DVD Rental Co who have been tasked with **generating the necessary data points** required to populate specific parts of this first-ever **customer email campaign**.

### Business Deliverables

The Marketing team have shared with us a draft of the email they wish to send to their customers.



We have summarized the data points to 7 parts.

Data Points | Email Item | What information do we need to find? | Flag Out to Marketing Team | 
----------- | ----------- | ------------------------------------ | -------------- |
1 & 4       | Top 2 Categories | Identify top 2 categories for each customer based off their past rental history. | - |
2 & 5       | Individual Customer Insights | For 1st category (2), identify total films watched, average comparison and percentile. For 2nd category (5), identify total films watched and proportion of films watched in percentage. | - |
3 & 6       | Category Film Recommendations | Identify 3 most popular films for each customer's top 2 categories that customer has not watched. | If customer does not have any film recommendations for either category. |
7, 8 &      | Favourite Actor Recommendation | Identify favourite actor with 3 film recommendations that have not been recommended. | If customer does not have at least 1 recommendation. |

***

## 📌 2.0 Entity Relationship Diagram
  
<img width="773" alt="1" src="https://github.com/gnash-k/newdata-project/assets/155973291/e0dca901-d6c3-4891-a755-0d7f116ada5a">


***

## 📌 3.0 Data Exploration

Before we dive into problem-solving, let's explore the data!

### 3.1 Validating Data with Hypotheses

We will develop a few hypotheses and test them using SQL.

#### 3.1.1 Hypothesis 1

> The number of unique `inventory_id` records will be equal in both `dvd_rentals.rentals` and `dvd_rentals.inventory` tables.

````sql
SELECT 
  COUNT(DISTINCT inventory_id) -- As there are multiple inventory_id for each film_id, run DISTINCT to find unique inventory_id
FROM dvd_rentals.rental
````
<img width="83" alt="2" src="https://github.com/gnash-k/newdata-project/assets/155973291/1c3fc078-f468-4b39-b00a-efb18592295a">


````sql
SELECT 
  COUNT(inventory_id)
FROM dvd_rentals.inventory;
````
<img width="91" alt="3" src="https://github.com/gnash-k/newdata-project/assets/155973291/b7ef0d5b-7ed1-4e15-8fe4-bbc580f4369c">


Looks like there are 4,580 `inventory_id` records in `dvd_rentals.rentals` and 4,581 `inventory_id` records in `dvd_rentals.inventory`. There is an additional 1 `inventory_id` record in `dvd_rentals.inventory`. 

#### 3.1.2 Hypothesis 2

> There will be a multiple records per unique `inventory_id` in the `dvd_rentals.rental` table.

````sql
WITH inventory_cte AS ( -- Generate a CTE using GROUP BY count of inventory_id
SELECT 
  inventory_id, 
  COUNT(*) AS inventory_id_count
FROM dvd_rentals.rental
GROUP BY inventory_id)
````
<img width="299" alt="4" src="https://github.com/gnash-k/newdata-project/assets/155973291/fb2b715d-fa8f-43d5-b4d0-21b1142a1047">



The table above shows the number of inventory records for each unique `inventory_id`. Then, we create a `CTE` and perform another grouping below.

````sql
SELECT 
  inventory_id_count,
  COUNT(*) AS inventory_id_grouping
FROM inventory_cte
GROUP BY inventory_id_count
ORDER BY inventory_id_count;
````

<img width="367" alt="5" src="https://github.com/gnash-k/newdata-project/assets/155973291/7aac77f1-56d2-44ab-ade6-a4a27203b22e">


Ok, `inventory_id_count` represents the number of inventory records for each film and `inventory_id_grouping` is `inventory_id_count` grouped by the number of records.

For example, in 1st row, there is 1 inventory_id/ film that has 4 copies of inventory/film.

#### 3.1.3 Hypothesis 3
> There will be multiple `inventory_id` records per unique `film_id` value in the `dvd_rentals.inventory` table

````sql
WITH inventory_grouped AS (
SELECT 
  DISTINCT(film_id) AS unique_film_id, 
  COUNT(*) AS inventory_records_count
FROM dvd_rentals.inventory
GROUP BY film_id)
````
<img width="291" alt="6" src="https://github.com/gnash-k/newdata-project/assets/155973291/353acd9a-5664-4889-8124-c902be382590">


````sql
SELECT 
  inventory_records_count, 
  COUNT(*)
FROM inventory_grouped
GROUP BY inventory_records_count
ORDER BY inventory_records_count;
````

<img width="283" alt="7" src="https://github.com/gnash-k/newdata-project/assets/155973291/5894b59d-3cbd-46d1-aa4f-1d2478881075">


***

### 3.2 Foreign Key Overlap Analysis

#### 3.2.1 Rental and Inventory Table
Let's revisit the following findings,

> "Looks like there are 4,580 `inventory_id` records in `dvd_rentals.rentals` and 4,581 `inventory_id` records in `dvd_rentals.inventory`. There is an additional 1 `inventory_id` record in `dvd_rentals.inventory`."

We will perform an ANTI JOIN on `dvd_rentals.inventory` to find out what is the additional record.

First, let's confirm our hypothesis one more time.

````sql
SELECT
  COUNT(DISTINCT i.inventory_id)
FROM dvd_rentals.inventory AS i
WHERE NOT EXISTS (
  SELECT r.inventory_id
  FROM dvd_rentals.rental AS r
  WHERE i.inventory_id = r.inventory_id);
````

<img width="112" alt="8" src="https://github.com/gnash-k/newdata-project/assets/155973291/bf249c3c-512e-4615-a68b-ac77dc9545e2">


````sql
SELECT
  *
FROM dvd_rentals.inventory AS i
WHERE NOT EXISTS (
  SELECT r.inventory_id
  FROM dvd_rentals.rental AS r
  WHERE i.inventory_id = r.inventory_id);
````

<img width="772" alt="9" src="https://github.com/gnash-k/newdata-project/assets/155973291/36c56fe5-fa80-4369-b2e5-5bd532490908">


We can make our assumption that this specific film is never rented by any customer, that's why it did not exist in the `rental` table which records only rental transactions. 

### 3.2.2 Inventory and Film Tables

Now, we will find out the relationship between `inventory` and `film` tables.

````sql
-- Find the number of unique film_id
SELECT 
  COUNT(DISTINCT film_id)
FROM dvd_rentals.film
````

<img width="95" alt="10" src="https://github.com/gnash-k/newdata-project/assets/155973291/d471e0f7-9e05-4b8b-91e9-ae0cd19d5c86">


There are 1,000 unique `film_id` records in `film` table.

````sql
-- Running ANTI JOIN to find out matching film_id records in both tables
SELECT 
  COUNT(*)
FROM dvd_rentals.film f
WHERE EXISTS
  (SELECT film_id
  FROM dvd_rentals.inventory i
  WHERE f.film_id = i.film_id)
````

<img width="90" alt="11" src="https://github.com/gnash-k/newdata-project/assets/155973291/ebd2f52f-5eeb-41e9-a21a-968a09066c01">


We will expect 958 records when we perform INNER JOIN between both tables.

***

## 4.0 Joining Tables

### joint_table table
````sql
DROP TABLE IF EXISTS joint_table;

CREATE TEMP TABLE joint_table AS
SELECT
  r.customer_id,
  i.film_id,
  f.title, 
  fc.category_id,
  c.name AS category_name,
  r.rental_date
FROM dvd_rentals.rental AS r
JOIN dvd_rentals.inventory AS i
  ON r.inventory_id = i.inventory_id
JOIN dvd_rentals.film AS f
  ON i.film_id = f.film_id
JOIN dvd_rentals.film_category AS fc
  ON f.film_id = fc.film_id
JOIN dvd_rentals.category AS c
  ON fc.category_id = c.category_id;
  
SELECT *
FROM joint_table 
WHERE customer_id = 1
ORDER BY rental_date DESC
LIMIT 5;
````

<img width="983" alt="12" src="https://github.com/gnash-k/newdata-project/assets/155973291/f054f49c-a862-4dbd-a6d3-965a49ec1b7e">


### category_rental_counts table

````sql
DROP TABLE IF EXISTS category_rental_counts;

CREATE TEMP TABLE category_rental_counts AS
SELECT 
  customer_id, 
  category_name, 
  COUNT(*) AS rental_count, 
  MAX(rental_date) AS latest_rental_date
FROM joint_table
GROUP BY customer_id, category_name;

SELECT *
FROM category_rental_counts 
WHERE customer_id = 1
ORDER BY rental_count DESC
LIMIT 5;
````

<img width="657" alt="13" src="https://github.com/gnash-k/newdata-project/assets/155973291/3b075637-ff03-46ad-8ad4-5250df2ce897">


### customer_total_rentals table

category_percentage: What proportion of each customer’s total films watched does this count make?

````sql
DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

SELECT *
FROM customer_total_rentals
LIMIT 5;
````

<img width="326" alt="14" src="https://github.com/gnash-k/newdata-project/assets/155973291/2810c04e-29f7-4443-a066-b30f95cd3941">


###  average_category_rental_counts table

````sql
DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  CONCAT_WS(' ', category_name, 'Category') AS category_name,
  FLOOR(AVG(rental_count)) AS avg_rental_count
FROM category_rental_counts
GROUP BY category_name;

SELECT *
FROM average_category_rental_counts
LIMIT 5;

````

<img width="349" alt="15" src="https://github.com/gnash-k/newdata-project/assets/155973291/dc90e5b8-8c95-40f3-b638-0b302d817e49">


###  customer_category_percentiles table

percentile: How does the customer rank in terms of the top X% compared to all other customers in this film category?

````sql
DROP TABLE IF EXISTS customer_category_percentiles;

CREATE TEMP TABLE customer_category_percentiles AS (
SELECT 
  customer_id, 
  category_name,
  CEILING(100 * 
    PERCENT_RANK() OVER (PARTITION BY category_name ORDER BY rental_count DESC)) as percentile
FROM category_rental_counts);

SELECT *
FROM customer_category_percentiles
WHERE customer_id = 1
ORDER BY percentile
LIMIT 5;
````

<img width="443" alt="16" src="https://github.com/gnash-k/newdata-project/assets/155973291/4ae9e634-63fc-4eb9-9a44-fbe041f0f827">


## Joining All Temporary Tables

````sql
DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile
FROM category_rental_counts AS t1
JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;
````

<img width="750" alt="17" src="https://github.com/gnash-k/newdata-project/assets/155973291/3f81e1c7-ecfa-4eb9-9af2-877eeaf6dcae">


## ✅ Learning Outcomes

<details>
<summary>
Click to view my learning outcomes!
  
</summary>
  
The following SQL skills and concepts will be covered in this section of the Serious SQL course:

1. Learning how to interpret ERDs for data literacy and business context (entity-relationship diagrams)
- Identify key columns of interest and how they are linked to other tables via foreign keys
- Use ERDs to analyze the data types for different columns in database tables
- Understand data context for various tables that cause inherent natural relationships between fields

2. Introduction to all types of table joining operations
- Simple joins: left, inner
- More involved joins: cross, anti, left-semi, full outer
- Combination set operations: union, union all, intersect, except

3. Practical application of table joins
- Joining multiple tables to combine disparate datasets into a single data structure
- Joining interim SQL outputs for more advanced group-by, split, merge data hacking strategies
- Performing table joins that use 2 or more table references in the ON condition
- Using anti joins to exclude records in a sequential fashion

4. Filtering, window functions and aggregates for analytics and ranking
- Using ROW_NUMBER to effecively rank order records with equal ties
- Using WHERE filters to extract ranked records
- Using multiple aggregate functions with different target partitions and ordering expressions for efficient data analysis
- Using aggregate group by clauses to generate unique customer level insights

5. Case statements for data transformation and manipulation
- Pivoting datasets from long to wide using MAX and CASE WHEN
- Manipulating actual data values using conditional logic for business translation purposes

6. SQL scripting basics
- Designing SQL workflows which can be easily understood and implemented
- Managing multiple dependencies for downstream table joining operations by using temporary tables to store interim datasets

7. Manipulating text data
- Converting text columns to title case
- Combining multiple text and numeric data type columns into a single text expression

</details>
  
***
