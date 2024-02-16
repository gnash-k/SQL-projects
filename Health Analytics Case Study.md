# 👩🏻‍⚕️ Health Analytics Mini Case Study

## 📌 Solution

### 1. How many unique users exist in the logs dataset?

````sql
SELECT 
  COUNT(DISTINCT id) AS unique_users
FROM health.user_logs
````
**Answer:**

<img width="141" alt="1" src="https://github.com/gnash-k/SQL-projects/assets/155973291/5c142f9a-1ada-4833-8c0e-a6e28b3c1ddb">

There are 554 unique users in the dataset.

***

### To answer Q2 to Q8, we will create a temporary table.

````sql
DROP TABLE IF EXISTS user_measure_count;

CREATE TEMP TABLE user_measure_count AS(
SELECT
  id,
  COUNT(*) AS measure_count,
  COUNT(DISTINCT measure) AS unique_measure_count
FROM health.user_logs
GROUP BY id);
````

<img width="801" alt="2" src="https://github.com/gnash-k/SQL-projects/assets/155973291/a2d601b0-2614-4c3e-bd43-706159ed9107">


Alright, once we have created the temp table, let's move on to our questions.

***

### 2. How many total measurements do we have per user on average?

Question is asking for the **average number of measurements for all users**.

````sql
SELECT 
  ROUND(AVG(measure_count),2) AS avg_measurement
FROM user_measure_count;
````

**Answer:**

<img width="161" alt="3" src="https://github.com/gnash-k/SQL-projects/assets/155973291/e63caf78-d3eb-46ec-a738-5dc0f43fc972">


The average measurements per user is 79 measurements. 

***

### 3. What about the median number of measurements per user?

````sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
````
**Answer:**

<img width="134" alt="4" src="https://github.com/gnash-k/SQL-projects/assets/155973291/94f8555f-3c10-4e7d-938d-0acf3b3511ec">


The median measurement per user is 2. 

***

### 4. How many users have 3 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3
````
**Answer:**

<img width="105" alt="5" src="https://github.com/gnash-k/SQL-projects/assets/155973291/99f18c35-f34a-40a2-89dd-ae0a2784817d">


209 users have 3 or more measurements.

***

### 5. How many users have 1,000 or more measurements?

````sql
SELECT 
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000
````
**Answer:**

<img width="98" alt="6" src="https://github.com/gnash-k/SQL-projects/assets/155973291/329bc081-c177-44df-9ed0-209fdbc867b0">


There are 5 users with 1,000 or more measurements.

***
### Looking at the logs data -
### 6. What is the number and percentage of the active user base who have logged blood glucose measurements?

````sql
SELECT
  measure,
  COUNT(DISTINCT id) AS unique_blood_glucose_user,
  ROUND(100 * COUNT(DISTINCT id)::NUMERIC / SUM(COUNT(DISTINCT id)) OVER (),2) AS blood_glucose_percentage
FROM health.user_logs
GROUP BY measure;
````
**Answer:**

<img width="750" alt="7" src="https://github.com/gnash-k/SQL-projects/assets/155973291/4f8945ba-24d1-4a26-8292-15c60263457a">

There are 325 users or 40% of user base who have logged blood glucose measurements.
***
### 7. What is the number and percentage of the active user base who have at least 2 types of measurements?

````sql
WITH measure_more_than_2 AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count >= 2)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(100 * COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN measure_more_than_2 AS m
  ON u.id = m.id;
````

**Answer:**

<img width="423" alt="8" src="https://github.com/gnash-k/SQL-projects/assets/155973291/ae271f6d-2433-4028-af5f-4188444168f8">


Out of 554 active users, 204 users have at least 2 types of measurements making up 37% of total active user base.
***
### 8. What is the number and percentage of the active user base who have all 3 measures - blood glucose, weight and blood pressure?

- First thing we will do is to create a `CTE` with results filtered to users with all measures.
- Then, we perform a `LEFT JOIN` on the user_measure_count and all_measures `CTE` meaning, we will retrieve all records in user_measure_count and matching records from all_measures `CTE`. 

````sql
WITH all_measures AS (
SELECT *
FROM user_measure_count
WHERE unique_measure_count = 3)

SELECT
  COUNT(DISTINCT m.id) AS unique_user,
  ROUND(COUNT(DISTINCT m.id)::numeric / COUNT(DISTINCT u.id),2) AS unique_user_percentage
FROM user_measure_count AS u
LEFT JOIN all_measures AS m
  ON u.id = m.id;
````
**Answer:**

<img width="434" alt="9" src="https://github.com/gnash-k/SQL-projects/assets/155973291/4ffc7f5e-a8ad-4ede-a724-6c6595d6dd47">


Out of 554 active users, 50 users have taken all 3 measures making up 9% of total active user base.
***
### 9. For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?

- First, we filter results to users with blood pressure measurements.
- Then, we find the median of systolic and diastolic measurements.

````sql
SELECT
  'blood_pressure' AS measure_name,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS systolic_median,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS diastolic_median
FROM health.user_logs
WHERE measure = 'blood_pressure';
````
**Answer:**

<img width="721" alt="10" src="https://github.com/gnash-k/SQL-projects/assets/155973291/d9f79780-a4ab-416f-a195-9f1e5fd0aaa6">

The median for systolic and diastolic are 126 and 79.

***
