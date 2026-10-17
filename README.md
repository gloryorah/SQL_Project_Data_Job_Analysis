# Introduction
Focusing on data analyst roles, this project explores top-paying jobs, in demand skills, and where high demand meets high salary in data analytics.

Check out the SQL queries here [project_sql folder](/project_sql/)

# Background
Driven by a quest to navigate the data analyst job market more effectively, this project was born from a desire to pinpoint top-paid and in-demand skills, streamlining others work to find optimal jobs.

Data hails from [SQL Course](https://lukebarousse.com/sql). It is packed with insights on job titles, salaries, locations, and essential skills.

### The questions I wanted to answer through my SQL queries were: 

1. What are the top=paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are the most in-demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn.

# Tools I used
For my deep dive into the data analyst job market, I harnessed the power of several key tools:

- **SQL:** The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL:** The chosen database management system, ideal for handling the job posting data.
- **Visual Studio Code:** My go-to for database management and executing SQL queries.
- **Git & GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.

# The analysis
Each query for this project aimed at investigating specific aspects of the data analyst job market. Here's how I approached each question:

### 1. Top Paying Data Analyst Jobs
To identify the highest paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.

```sql
SELECT
	job_id,
	job_title,
	job_location,
	job_schedule_type,
	salary_year_avg,
	job_posted_date,
	name AS company_name
FROM
	job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
	job_title_short = 'Data Analyst' AND
	job_location = 'Anywhere' AND
	salary_year_avg IS NOT NULL
ORDER BY
	salary_year_avg desc
LIMIT 10
```
Here's the breakdown of the top data analyst jobs in 2023:
- **Wide Salary Range:** Top 10 paying data analyst roles span from $184,000 to $650,000, indicating significant salary potential in the field.
- **Diverse Employers:** Companies like SmartAsset, Meta, and AT&T are among those offering high salaries, showing a broad interest among different industries.
- **Job Title Variety:** There is a high diversity in job titles, from Data Analyst to Director of Analytics, reflecting varied roles and specializations within data analytics.

### 2. Skills Required for Top-Paying Data Analyst Jobs

To identify the skills required for the top-paying data analyst jobs, I used the results from the first query and joined the job postings data with the skills data to identify the specific skills required for each role. This query provides a detailed look at the skills associated with high-paying jobs, helping job seekers understand which skills to develop to align with top salaries.

```sql
WITH top_paying_jobs AS (
	SELECT
		job_id,
		job_title,
		salary_year_avg,
		name AS company_name
	FROM
		job_postings_fact
	LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
	WHERE
		job_title_short = 'Data Analyst' AND
		job_location = 'Anywhere' AND
		salary_year_avg IS NOT NULL
	ORDER BY
		salary_year_avg DESC
)
SELECT 
	top_paying_jobs.*,
	skills 
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
	salary_year_avg DESC
```

Here's a breakdown of the skills required for the top-paying data analyst jobs:

- **Wide Range of Skills:** The roles require a variety of technical skills, including SQL, Python, R, Azure, Databricks, AWS, and PySpark.

- **Data Visualization Tools:** Tableau and Power BI appear among the required skills, alongside Excel and PowerPoint.

- **Specialized Technologies:** Other requirements include Pandas, Jupyter, Hadoop, Oracle, Crystal, and Flow, depending on the specific role.

### 3. Most In-Demand Skills for Data Analysts

To identify the most in-demand skills for data analysts, I joined the job postings data with the skills data and counted how many job postings required each skill. I focused on all data analyst job postings to identify the top five skills with the highest demand in the job market, providing insight into the most valuable skills for job seekers.

```sql
SELECT 
	skills,
	count (skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE 
	job_title_short = 'Data Analyst'
GROUP BY
	skills
ORDER BY demand_count DESC
LIMIT 5
```

Here's a breakdown of the most in-demand skills for data analysts in 2023, based on job postings:

- **SQL:** Leads the list with a demand count of 8.
- **Python:** Follows closely with a demand count of 7.
- **Tableau:** Is also highly sought after, with a demand count of 6.
- **Other Skills:** R, Snowflake, Pandas, and Excel also show varying degrees of demand.

### 4. Top Skills Based on Salary

To identify the top skills based on salary, I calculated the average yearly salary associated with each skill for data analyst positions. I focused on remote roles with specified salaries, regardless of location, to identify the skills associated with higher-paying opportunities and help job seekers identify financially rewarding skills to acquire or improve.

```sql
SELECT 
	skills,
	ROUND(AVG (salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE 
	job_title_short = 'Data Analyst' AND
	salary_year_avg IS NOT NULL
	AND job_work_from_home = True
GROUP BY
	skills
ORDER BY avg_salary DESC
LIMIT 25
```

Here's a breakdown of the top skills based on average yearly salary for remote data analyst positions:

- **PySpark:** Ranks first, with an average yearly salary of $208,172.
- **Bitbucket:** Follows with an average yearly salary of $189,155.
- **Couchbase and Watson:** Both have an average yearly salary of $160,515.
- **Other High-Paying Skills:** DataRobot, GitLab, Swift, Jupyter, and Pandas also feature among the top 10 skills, with average yearly salaries ranging from $151,821 to $155,486.

### 5. Most Optimal Skills to Learn

To identify the most optimal skills to learn, I combined skill demand counts with average yearly salaries for data analyst positions. I focused on remote positions with specified salaries and filtered for skills appearing in more than 10 job postings. This query helps identify skills that offer both job opportunities and financial benefits, providing strategic insights for career development in data analytics.

```sql
WITH skills_demand AS (
	SELECT 
		skills_dim.skill_id,
		skills_dim.skills,
		COUNT (skills_job_dim.job_id) AS demand_count
	FROM job_postings_fact
	INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
	WHERE 
		job_title_short = 'Data Analyst'
		AND salary_year_avg IS NOT NULL
		AND job_work_from_home = True
	GROUP BY
		skills_dim.skill_id
), average_salary AS (
	SELECT 
		skills_job_dim.skill_id,
		ROUND(AVG (salary_year_avg), 0) AS avg_salary
	FROM job_postings_fact
	INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
	WHERE 
		job_title_short = 'Data Analyst' AND
		salary_year_avg IS NOT NULL
		AND job_work_from_home = True
	GROUP BY
		skills_job_dim.skill_ID
)

SELECT
	skills_demand.skill_id,
	skills_demand.skills,
	demand_count,
	avg_salary
FROM 
	skills_demand 
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE 
	demand_count>10
ORDER BY 
	avg_salary DESC,
	demand_count DESC 
LIMIT 25
```

Here's a breakdown of skills with high demand and average yearly salaries for remote data analyst positions:

- **Go:** Leads the list with an average yearly salary of $115,320 and a demand count of 27.
- **Snowflake:** Has a demand count of 37 and an average yearly salary of $112,948.
- **Azure:** Shows a demand count of 34 and an average yearly salary of $111,225.
- **Python:** Stands out with a demand count of 236 and an average yearly salary of $101,397.
- **Tableau:** Has a demand count of 230 and an average yearly salary of $99,288.
- **Other Skills:** Hadoop, AWS, Oracle, Looker, and R also feature among the skills with notable demand and average yearly salaries.

# What I learned

Through this project, I strengthened my SQL skills and gained insights into the data analyst job market. Here are the key things I learned:

- **Complex Query Crafting:** Improved my ability to write advanced SQL queries, join tables, and use `WITH` clauses for Common Table Expressions (CTEs).
- **Data Aggregation:** Became more comfortable using `GROUP BY` and aggregate functions like `COUNT()` and `AVG()` to summarise data.
- **Analytical Mastery:** Improved my real-world problem-solving skills by translating questions into SQL queries that uncover useful insights.

# Conclusions

### Insights
From the analysis, several general insights emerged:

1. **Top-Paying Data Analyst Jobs**: The highest-paying jobs for data analysts that allow remote work offer a wide range of salaries, the highest at $650,000!
2. **Skills for Top-Paying Jobs**: High-paying data analyst jobs require advanced proficiency in SQL, suggesting it's a critical skill for earning a top salary.
3. **Most In-Demand Skills**: SQL is also the most demanded skill in the data analyst job market, thus making it essential for job seekers.
4. **Skills with Higher Salaries**: Specialized skills, such as SVN and Solidity, are associated with the highest average salaries, indicating a premium on niche expertise.
5. **Optimal Skills for Job Market Value**: SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value.

### Closing Thoughts

This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.
