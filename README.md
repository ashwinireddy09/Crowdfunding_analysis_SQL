# Crowdfunding_analysis_SQL
Crowdfunding Data Analysis project using MySQL to explore project trends, success rates, backers, and funding goals across countries and categories. Includes data cleaning, exploratory data analysis (EDA), and conversion of funding amounts to USD. Designed to extract meaningful business insights from real-world crowdfunding data.

Project Overview
Project Title: Crowdfunding Projects Analysis
Level: Beginner to Intermediate
Database: cf_analysis

This project demonstrates SQL skills and techniques used by data analysts to explore, clean, and analyze real-world crowdfunding project data. The project involves creating a database, performing data cleaning, conducting exploratory data analysis, and answering key business questions using SQL.
This project is ideal for beginners who want hands-on practice with SQL using a real business dataset.

## Objectives
- Convert Epoch time to readable dates.
- Build a Calendar Table with multiple date-related attributes.
- Build a data model using project datasets.
- Convert goal amounts into USD using a static rate.
- Analyze project performance (outcome, location, category, timeline).
- Identify top successful projects.
- Calculate percentages of successful projects overall, by category, by time, and by goal range.

## Project Structure

- **Database Creation**: The project starts by creating a database named `cf_analysis`.
- **Table Creation**: A table named `crowdfunding_projects1` is created to store restaurant-related data. The table structure includes columns for restaurant ID, restaurant name, country, city, cuisine type, average cost for two, currency, rating, votes, online delivery availability, price range, and opening date.

# 1. Database and Table Creation
```sql
CREATE DATABASE cf_analysis;

USE cf_analysis;

CREATE TABLE crowdfunding_projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(255),
    creator_name VARCHAR(100),
    category VARCHAR(50),
    sub_category VARCHAR(50),
    country VARCHAR(50),
    city VARCHAR(50),
    goal_amount DECIMAL(15,2),
    currency VARCHAR(10),
    amount_raised DECIMAL(15,2),
    backers_count INT,
    created_date_epoch BIGINT,
    launched_date_epoch BIGINT,
    deadline_date_epoch BIGINT,
    outcome VARCHAR(20)
);

```
# 2. Convert Epoch Time to Natural Dates

```sql
  
-- Convert created date
ALTER TABLE crowdfunding_projects
ADD COLUMN created_date DATE,
ADD COLUMN launched_date DATE,
ADD COLUMN deadline_date DATE;

UPDATE crowdfunding_projects
SET created_date = FROM_UNIXTIME(created_date_epoch),
    launched_date = FROM_UNIXTIME(launched_date_epoch),
    deadline_date = FROM_UNIXTIME(deadline_date_epoch);

```
# 3. Build Calendar Table
1. Convert Goal and Pledged Amounts into USD

```sql
   SELECT 
    project_id,
    project_name,
    country,
    goal_amount,
    pledged_amount,
    ROUND(goal_amount / 83, 2) AS goal_usd,
    ROUND(pledged_amount / 83, 2) AS pledged_usd
FROM crowdfunding_projects;
```
2. Count of Projects by Country and Category

```sql
CREATE TABLE calendar_table (
    calendar_date DATE PRIMARY KEY,
    year INT,
    month_no INT,
    month_name VARCHAR(20),
    quarter VARCHAR(2),
    year_month VARCHAR(10),
    weekday_no INT,
    weekday_name VARCHAR(20),
    financial_month VARCHAR(5),
    financial_quarter VARCHAR(5)
);

-- Populate calendar table (example for a range)
INSERT INTO calendar_table (calendar_date, year, month_no, month_name, quarter, year_month, weekday_no, weekday_name, financial_month, financial_quarter)
SELECT 
    cal_date,
    YEAR(cal_date),
    MONTH(cal_date),
    DATE_FORMAT(cal_date,'%M'),
    CONCAT('Q',QUARTER(cal_date)),
    DATE_FORMAT(cal_date,'%Y-%b'),
    DAYOFWEEK(cal_date),
    DATE_FORMAT(cal_date,'%W'),
    CONCAT('FM', CASE WHEN MONTH(cal_date) >=4 THEN MONTH(cal_date)-3 ELSE MONTH(cal_date)+9 END),
    CONCAT('FQ', CASE WHEN MONTH(cal_date) >=4 THEN QUARTER(DATE_SUB(cal_date, INTERVAL 3 MONTH)) ELSE QUARTER(DATE_ADD(cal_date, INTERVAL 9 MONTH)) END)
FROM (
    SELECT ADDDATE('2009-01-01', INTERVAL t.n DAY) AS cal_date
    FROM (SELECT a.N + b.N * 10 + c.N * 100 AS n
          FROM (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) a
          CROSS JOIN (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) b
          CROSS JOIN (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) c
         ) t
    WHERE ADDDATE('2009-01-01', INTERVAL t.n DAY) <= '2030-12-31'
) calendar_dates;

```
4. Projects Created by Year, Quarter, Month

```sql
SELECT 
    YEAR(created_at) AS year,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY year
ORDER BY year;

SELECT 
    YEAR(created_at) AS year,
    QUARTER(created_at) AS quarter,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY year, quarter
ORDER BY year, quarter;

SELECT 
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY year, month
ORDER BY year, month;
```
4. Convert Goal Amount into USD
```sql
-- Assuming static USD rate = 83
ALTER TABLE crowdfunding_projects ADD COLUMN goal_usd DECIMAL(15,2);

UPDATE crowdfunding_projects
SET goal_usd = ROUND(goal_amount / 83,2);


```
5. Projects Overview KPIs
```sql
-- Total Projects by Outcome
SELECT outcome, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY outcome;

-- Total Projects by Location (Country & City)
SELECT country, city, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY country, city
ORDER BY total_projects DESC;

-- Total Projects by Category
SELECT category, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY category
ORDER BY total_projects DESC;

-- Projects Created by Year, Quarter, Month
SELECT YEAR(created_date) AS year, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY year
ORDER BY year;

SELECT YEAR(created_date) AS year, QUARTER(created_date) AS quarter, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY year, quarter
ORDER BY year, quarter;

SELECT YEAR(created_date) AS year, MONTH(created_date) AS month, COUNT(*) AS total_projects
FROM crowdfunding_projects
GROUP BY year, month
ORDER BY year, month;

```
6. Successful Projects Analysis

```sql
-- Overall Metrics for Successful Projects
SELECT 
    COUNT(*) AS total_successful_projects,
    SUM(amount_raised) AS total_amount_raised,
    SUM(backers_count) AS total_backers,
    ROUND(AVG(DATEDIFF(deadline_date, launched_date)),2) AS avg_days_to_success
FROM crowdfunding_projects
WHERE outcome='successful';

```
7. Top Successful Projects
```sql
-- Top by Number of Backers
SELECT project_name, backers_count, amount_raised
FROM crowdfunding_projects
WHERE outcome='successful'
ORDER BY backers_count DESC
LIMIT 10;

-- Top by Amount Raised
SELECT project_name, amount_raised, backers_count
FROM crowdfunding_projects
WHERE outcome='successful'
ORDER BY amount_raised DESC
LIMIT 10;

```
8. Percentage Analysis

```sql
-- Percentage of Successful Projects Overall
SELECT ROUND(SUM(CASE WHEN outcome='successful' THEN 1 ELSE 0 END)*100/COUNT(*),2) AS success_percentage
FROM crowdfunding_projects;

-- Percentage by Category
SELECT category,
       ROUND(SUM(CASE WHEN outcome='successful' THEN 1 ELSE 0 END)*100/COUNT(*),2) AS success_percentage
FROM crowdfunding_projects
GROUP BY category;

-- Percentage by Year
SELECT YEAR(created_date) AS year,
       ROUND(SUM(CASE WHEN outcome='successful' THEN 1 ELSE 0 END)*100/COUNT(*),2) AS success_percentage
FROM crowdfunding_projects
GROUP BY year
ORDER BY year;

-- Percentage by Goal Range
SELECT 
    CASE 
        WHEN goal_usd < 500 THEN 'Low (<500)'
        WHEN goal_usd BETWEEN 500 AND 2000 THEN 'Medium (500-2000)'
        WHEN goal_usd BETWEEN 2001 AND 5000 THEN 'High (2001-5000)'
        ELSE 'Very High (>5000)'
    END AS goal_range,
    ROUND(SUM(CASE WHEN outcome='successful' THEN 1 ELSE 0 END)*100/COUNT(*),2) AS success_percentage
FROM crowdfunding_projects
GROUP BY goal_range;


```
# Findings / Reports

- Project Performance Report: Based on amount raised, backers, and success rate.
- Category Insights: Identify which categories have the highest success rates.
- Timeline Insights: Evaluate projects launched over time by year/month/quarter.
- Goal vs Success: Determine optimal goal ranges for higher success probability.
- Top Projects Report: Highlight most funded and most backed projects.

# Conclusion

This Crowdfunding Analysis SQL Project demonstrates how to clean, transform, and analyze real-world project data. It provides insights into project success trends, funding patterns, top projects, and KPIs across categories, locations, and time—showcasing practical SQL skills for data-driven decision making.
