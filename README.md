# Student-Social-Media-Addiction-Analysis-Power-BI-Dashboard
This project analyzes how student social media usage relates to sleep, academic impact, and mental health. The interactive Power BI dashboard visualizes trends and key metrics to support data-driven conclusions.


Project

Student Social Media Addiction Analysis — Power BI Dashboard

Overview

This project analyzes how student social media usage relates to sleep, academic impact, and mental health. The interactive Power BI dashboard visualizes trends and key metrics to support data-driven conclusions.

Tools & Technologies

Power BI (dashboard & visuals)

Power Query (ETL / transformations)

DAX (measures & dynamic KPIs)

SQL (data extraction, cleaning, aggregation) — key role in the pipeline

CSV dataset (source)

Data Pipeline & How SQL Was Used

I used SQL as the first step to reliably extract and pre-aggregate data before bringing it into Power BI. Key SQL responsibilities:

Extract raw rows and filter bad records

Normalize/standardize categorical values (platform names, country names)

Aggregate usage and compute sample flags (e.g., HighUsage)

Join survey metadata if present

Produce slim, analytics-ready tables for Power BI

Example workflow:

Raw data stored in a table student_survey_raw.

Use SQL queries to clean and create student_survey_clean (analytics table).

Import cleaned table into Power BI (or export to CSV and then import).

Example SQL queries (real)

1) Create analytics table (normalized & filtered):

CREATE TABLE student_survey_clean AS
SELECT
  Student_ID,
  Age,
  Gender,
  Academic_Level,
  TRIM(LOWER(Country)) AS Country,
  Avg_Daily_Usage_Hours,
  LOWER(Most_Used_Platform) AS Most_Used_Platform,
  CASE WHEN Affects_Academic_Performance IN ('Yes','Y','yes') THEN 1 ELSE 0 END AS Academic_Impact_Flag,
  Sleep_Hours_Per_Night,
  Mental_Health_Score,
  Relationship_Status,
  Conflicts_Over_Social_Media,
  Addicted_Score
FROM student_survey_raw
WHERE Student_ID IS NOT NULL
  AND Avg_Daily_Usage_Hours IS NOT NULL;


2) Aggregate: average metrics by academic level

SELECT
  Academic_Level,
  ROUND(AVG(Avg_Daily_Usage_Hours),2) AS Avg_Hours,
  ROUND(AVG(Addicted_Score),2) AS Avg_Addicted_Score,
  ROUND(AVG(Mental_Health_Score),2) AS Avg_Mental_Health
FROM student_survey_clean
GROUP BY Academic_Level
ORDER BY Avg_Hours DESC;


3) Top platforms by total usage hours

SELECT
  Most_Used_Platform,
  SUM(Avg_Daily_Usage_Hours) AS Total_Usage_Hours,
  COUNT(*) AS Users
FROM student_survey_clean
GROUP BY Most_Used_Platform
ORDER BY Total_Usage_Hours DESC
LIMIT 10;


4) Create a high-usage flag with a CTE / window function (useful for drilldowns)

WITH usage_stats AS (
  SELECT
    Student_ID,
    Avg_Daily_Usage_Hours,
    CASE WHEN Avg_Daily_Usage_Hours > 3 THEN 1 ELSE 0 END AS High_Usage
  FROM student_survey_clean
)
SELECT
  High_Usage,
  COUNT(*) AS CountUsers,
  ROUND(AVG(Addicted_Score),2) AS AvgAddiction
FROM usage_stats u
JOIN student_survey_clean s USING (Student_ID)
GROUP BY High_Usage;


5) Country & gender breakdown (for the country/gender bar chart)

SELECT
  Country,
  Gender,
  COUNT(*) AS StudentCount
FROM student_survey_clean
GROUP BY Country, Gender
ORDER BY Country, StudentCount DESC;

Power BI: Visuals & Implementation Steps

Import cleaned SQL table (or CSV exported from SQL).

Power Query: final transformations (data types, rename columns, create AgeGroup column).

Model & Measures (DAX) — examples:

Avg_Daily_Usage = AVERAGE('student_survey_clean'[Avg_Daily_Usage_Hours])
Avg_Addicted_Score = AVERAGE('student_survey_clean'[Addicted_Score])
Academic_Impact_Percent = DIVIDE(CALCULATE(COUNTROWS('student_survey_clean'), 'student_survey_clean'[Academic_Impact_Flag] = 1), COUNTROWS('student_survey_clean')) * 100


Build visuals:

KPI cards (Avg Hours, Avg Addiction Score, Avg Sleep, Academic Impact %)

Line chart (Addicted Score vs Mental Health by Academic_Level)

Column chart (Avg usage by Most_Used_Platform)

Pie/Donut chart (Gender distribution)

Country bar charts & Country x Gender clustered chart

Interactivity: add slicers (Country, Academic_Level, Gender) and enable cross-filtering and tooltips.

Key Insights (summary)

Avg. daily social media usage ~ 4.9 hrs/day.

Instagram leads in total usage hours.

Higher usage correlates with higher addiction score and lower sleep/mental-health scores.

Academic impact (%) is significant — useful for stakeholders (educators, counselors).

Best Practices & Notes on SQL usage

Do heavy joins, aggregations, and string-normalization in SQL — it reduces Power BI model complexity.

Use indexes on Student_ID, Country, and timestamp or platform fields for faster queries on large datasets.

Create views for frequently used aggregations (e.g., vw_platform_usage) to keep PBIX lightweight.

Author

Created by: Mr. Srikanth
Role: MBA (Finance) Graduate| Aspiring Data Analyst | Power BI & SQL Developer 
