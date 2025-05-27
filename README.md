# Hotel Operations Performance Analysis with PostgreSQL
A practical SQL case study on hotel operations performance and customer satisfaction.

📄 For the full analysis and storytelling behind this project, read the detailed Medium article:
👉 [Hotel Service Performance Analysis with PostgreSQL](https://medium.com/@UjuEmmanuella/hotel-service-performance-analysis-with-postgresql-5ad04c3dae36)

The article walks through the entire project, from data cleaning to insights and recommendations, showcasing how SQL was used to uncover service bottlenecks and quality issues across hotel branches.

## Project Overview
This project was part of a SQL assessment test for a role focused on data operations and analytics. The task involved working with hotel operations data from LuxurStay Hotels — a global chain with branches in EMEA, NA, LATAM, and APAC. The goal was to identify bottlenecks in service delivery, especially in branches where customer satisfaction had fallen below the expected 4.5-star rating.

I worked through a set of business tasks to clean and prepare the data, compute performance metrics, and generate actionable insights.

---

## Project Type
- SQL Data Cleaning & Manipulation

## Tools Used
- PostgreSQL
- Datalab


## Skills Demonstrated
Data Cleaning · Data Transformation · Aggregation · Joins · Conditional Logic · Case Statements

---

## Data Source
The dataset was drawn from LuxurStay Hotels’ operational database as part of SQL skills assessment, from DataCamp. The schema included data on:

- **branch**: Information about each hotel branch (e.g., location, number of rooms, staff, opening date).

- **request**: Records of customer service requests and associated performance data (e.g., time taken and customer ratings).

- **service**: Lookup table with service descriptions such as Meal, Laundry, etc.

The data only included records where customers had given feedback, making it suitable for customer satisfaction and service delivery analysis.

---

## Project Objective
LuxurStay Hotels has been experiencing a decline in customer satisfaction due to slow room service responses in several locations. Management noticed a drop from their expected average customer rating of 4.5 and wanted to understand the root causes.

As a Data Analyst supporting the Head of Operations, my task was to:

- Clean and validate hotel branch data to ensure its readiness for analysis.

- Identify performance issues in customer service delivery.

- Highlight underperforming hotel branches and services.

- Support decision-making to improve operations and restore satisfaction levels.

---

## Task 1: Data Cleaning & Standardization

The first challenge involved cleaning and standardizing the branch table. It was known to have missing values and inconsistent entries across multiple columns. I applied the following logic:

**Replaced Nulls & Defaults**:

- **location:** Filled missing values with **'Unknown'**.

- **total_rooms:** Validated range between **1–400**, defaulted to **100** if missing or invalid.

- **staff_count:** Estimated missing staff count by multiplying **total_rooms * 1.5.**

- **opening_date:** Replaced missing or invalid entries with **2023.**

- **target_guests:** Defaulted to **'Leisure'** when missing.

This process ensured the data met the schema expectations and was analysis-ready.

```Sql
SELECT 
    id,
    COALESCE(location, 'Unknown') AS location,
    CASE
        WHEN total_rooms BETWEEN 1 AND 400 THEN total_rooms
        ELSE 100
    END AS total_rooms,
    CASE
        WHEN staff_count IS NOT NULL THEN staff_count
        ELSE total_rooms * 1.5
    END AS staff_count,
    CASE
        WHEN opening_date = '-' THEN '2023'
        WHEN opening_date BETWEEN '2000' AND '2023' THEN opening_date
        ELSE '2023'
    END AS opening_date,
    CASE
        WHEN target_guests IS NULL THEN 'Leisure'
        WHEN LOWER(target_guests) LIKE 'b%' THEN 'Business'
        ELSE target_guests
    END AS target_guests
FROM 
    public.branch;
```

---

## Task 2: Branch-Level Performance Analysis

I calculated the average and maximum time taken per service and hotel branch. This helped identify which combinations were delivering services slower than others.


```sql
SELECT 
    service_id, 
    branch_id, 
    ROUND(AVG(time_taken), 2) AS avg_time_taken, 
    MAX(time_taken) AS max_time_taken
FROM public.request
GROUP BY service_id, branch_id;
```

---

## Task 3: Targeted Services & Regions
The management team wanted to focus on services with frequent complaints — **Meal** and **Laundry** — and **regions** flagged by management (EMEA & LATAM), I pulled relevant requests and their ratings to support a deeper performance audit. I filtered relevant records and retrieved:

- **Service description**

- **Branch ID** and **location**

- **Request ID**

- **Customer rating**

This was important for zooming in on high-impact areas.

```sql
SELECT
    s.description,
    b.id AS branch_id,
    b.location,
    r.id AS request_id,
    r.rating
FROM
    request r
    JOIN branch b ON b.id = r.branch_id
    JOIN service s ON r.service_id = s.id
WHERE
    s.description IN ('Meal', 'Laundry')
    AND b.location IN ('EMEA', 'LATAM');
```

---

## Task 4: Identify Underperforming Branches

To align with management’s target rating threshold of 4.5, I grouped requests by branch_id and service_id to find combinations falling below this benchmark. This allows the operations team to prioritize service and location improvements.

```sql
SELECT 
    service_id, 
    branch_id, 
    ROUND(AVG(rating), 2) AS avg_rating
FROM public.request
GROUP BY service_id, branch_id
HAVING AVG(rating) < 4.5;
```
---

## Key Insights

- Several branches had inconsistent or missing metadata—especially in locations and staff numbers.

- Some services (like Laundry in LATAM) showed consistently high response times.

- Multiple branch-service combinations had average ratings below 4.5, indicating that service quality wasn’t meeting standards.

---

## Recommendations

- Based on the analysis, I propose the following steps:

- Improve Data Entry Standards

- Clean up and validate branch metadata regularly to avoid inconsistencies that affect analysis and planning.

- Audit Slow Services

- Focus on high time-to-response services, especially in underperforming regions like EMEA and LATAM.

- Implement Training or Staffing Adjustments

- Use staff_count vs. total_rooms ratios to assess if branches are under-resourced, then reallocate or hire accordingly.

- Follow-Up Surveys

- Send follow-up surveys to customers who gave low ratings to understand the deeper causes behind dissatisfaction.

---

## 📂 Files & Resources

📜 SQL Script: ![luxurstay_operations&cleaning_analysis.sqlluxurstay_operations&cleaning_analysis.sql]

✍🏽 Medium Write-Up : Read article [here](https://medium.com/@UjuEmmanuella/hotel-service-performance-analysis-with-postgresql-5ad04c3dae36)
