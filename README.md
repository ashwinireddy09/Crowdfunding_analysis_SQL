# Crowdfunding_analysis_SQL
Crowdfunding Data Analysis project using MySQL to explore project trends, success rates, backers, and funding goals across countries and categories. Includes data cleaning, exploratory data analysis (EDA), and conversion of funding amounts to USD. Designed to extract meaningful business insights from real-world crowdfunding data.

Project Overview
Project Title: Crowdfunding Projects Analysis
Level: Beginner to Intermediate
Database: cf_analysis

This project demonstrates SQL skills and techniques used by data analysts to explore, clean, and analyze real-world crowdfunding project data. The project involves creating a database, performing data cleaning, conducting exploratory data analysis, and answering key business questions using SQL.
This project is ideal for beginners who want hands-on practice with SQL using a real business dataset.

## Objectives
- Set up the Crowdfunding database.
- Perform data cleaning and handle missing values.
- Conduct exploratory data analysis (EDA).
- Analyze project outcomes, backers, funding goals, and categories.
- Derive meaningful business insights using SQL queries.

## Project Structure

- **Database Creation**: The project starts by creating a database named `cf_analysis`.
- **Table Creation**: A table named `crowdfunding_projects1` is created to store restaurant-related data. The table structure includes columns for restaurant ID, restaurant name, country, city, cuisine type, average cost for two, currency, rating, votes, online delivery availability, price range, and opening date.

```sql
CREATE TABLE crowdfunding_projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(150),
    category VARCHAR(50),
    country VARCHAR(50),
    currency VARCHAR(10),
    goal_amount FLOAT,
    pledged_amount FLOAT,
    backers_count INT,
    status VARCHAR(20),
    created_at DATE
);
```
# 2. Data Exploration & Cleaning

- Record Count: Determine the total number of project records.
- Unique Project Count: Count unique projects.
- Category Count: Find all unique categories.
- Unique Country Count: Count how many countries are represented.
- Null Value Check: Identify missing or NULL values in critical columns.
- Delete Records with Missing Data: Remove rows with NULL values in key columns.

```sql
  
SELECT COUNT(*) FROM crowdfunding_projects;
SELECT COUNT(DISTINCT project_name) FROM crowdfunding_projects;

SELECT DISTINCT category FROM crowdfunding_projects;

SELECT COUNT(DISTINCT country) FROM crowdfunding_projects;

SELECT * 
FROM crowdfunding_projects
WHERE 
    project_name IS NULL OR category IS NULL OR country IS NULL OR
    goal_amount IS NULL OR pledged_amount IS NULL OR backers_count IS NULL;

DELETE FROM crowdfunding_projects
WHERE 
    project_name IS NULL OR category IS NULL OR country IS NULL OR
    goal_amount IS NULL OR pledged_amount IS NULL OR backers_count IS NULL;

```
# 3. Data Analysis & Findings
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
SELECT 
    country,
    category,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY country, category
ORDER BY total_projects DESC;
ORDER BY total_projects DESC;

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
4. Count of Projects Based on Status (Success, Failed, Canceled)
```sql
SELECT 
    status,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY status;

```
5. Create Funding Buckets Based on Goal Amount
```sql
SELECT 
    CASE 
        WHEN goal_amount < 1000 THEN 'Low (<1K USD)'
        WHEN goal_amount BETWEEN 1000 AND 5000 THEN 'Medium (1K-5K USD)'
        WHEN goal_amount BETWEEN 5001 AND 20000 THEN 'High (5K-20K USD)'
        ELSE 'Very High (>20K USD)'
    END AS goal_bucket,
    COUNT(project_id) AS total_projects
FROM crowdfunding_projects
GROUP BY goal_bucket
ORDER BY total_projects DESC;
```
6. Percentage of Projects by Status

```sql
SELECT 
    status,
    COUNT(*) AS total_projects,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM crowdfunding_projects), 2) AS percentage
FROM crowdfunding_projects
GROUP BY status;

```
7. Top Categories and Countries by Successful Projects
```sql
SELECT 
    category,
    COUNT(project_id) AS successful_projects
FROM crowdfunding_projects
WHERE status='successful'
GROUP BY category
ORDER BY successful_projects DESC;

SELECT 
    country,
    COUNT(project_id) AS successful_projects
FROM crowdfunding_projects
WHERE status='successful'
GROUP BY country
ORDER BY successful_projects DESC;
```
9. Build a KPI Dashboard

```sql
SELECT  
    COUNT(project_id) AS total_projects,
    SUM(CASE WHEN status='successful' THEN 1 ELSE 0 END) AS successful_projects,
    SUM(CASE WHEN status='failed' THEN 1 ELSE 0 END) AS failed_projects,
    ROUND(AVG(goal_amount),2) AS avg_goal,
    ROUND(AVG(pledged_amount),2) AS avg_pledged,
    SUM(backers_count) AS total_backers
FROM crowdfunding_projects;

```
# Findings

- Project Success Trends: Success rates vary by category and country.
- Popular Categories: Technology, Design, and Games dominate crowdfunding projects.
- Funding Goals: Most projects have medium-level goals (<$5,000).
- Backer Trends: Successful projects attract significantly more backers.
- Country Insights: US, UK, and Canada lead in the number of projects and successful campaigns.

# Reports
- Project Performance Report: Status, funding, and backers analysis.
- Category Insights: Success rate and average funding by category.
- Country-wise Insights: Country trends and funding performance.
- Funding Strategy: Low, medium, high, and very high goal segmentation.

# Conclusion

This Crowdfunding Analysis SQL Project provides hands-on learning experience in SQL, covering database setup, data cleaning, exploratory data analysis, and business-driven queries. The insights derived help understand project success factors, funding patterns, category performance, and country trends—making it highly relevant for real-world data analyst roles.

