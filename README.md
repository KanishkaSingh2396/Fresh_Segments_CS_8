# Fresh_Segments_CS_8

![image](https://user-images.githubusercontent.com/89623051/143212491-e4e753ed-fc7b-40ef-b653-b8406db93489.png)

## Introduction

A digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.


## Fresh Segment Tables

## Interest Metrics

This table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base.

Each record in this table represents the performance of a specific interest_id based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content.

![image](https://user-images.githubusercontent.com/89623051/143213285-96149b36-bb6a-45c6-b407-0eb1cde86a0e.png)

## Interest Map

This mapping table links the interest_id with their relevant interest information. You will need to join this table onto the previous interest_details table to obtain the interest_name as well as any details about the summary information.

![image](https://user-images.githubusercontent.com/89623051/143213512-63002497-805c-4add-a0fa-6f9555132a0a.png)

## Case Study Questions
The following questions can be considered key business questions that are required to be answered for the Fresh Segments team.

## A. Data Exploration and Cleansing

### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```sql

select to_date(month_year, 'MM YYYY') as month_year_date from fresh_segments.interest_metrics
```

![image](https://user-images.githubusercontent.com/89623051/143396119-d36e4f83-9cd7-4d0d-abcf-26b4cdd70c6b.png)

### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```sql
select count(*),  to_date(month_year, 'MM YYYY')  as month_year 
from fresh_segments.interest_metrics
where month_year is not null
group by month_year
order by month_year
```

![image](https://user-images.githubusercontent.com/89623051/143396220-970a9f25-5ec8-4b4c-ba7d-0b173b74e2e5.png)



### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```SQL
SELECT
  COUNT(DISTINCT interest_metrics.interest_id) AS all_interest_metric,
  COUNT(DISTINCT interest_map.id) AS all_interest_map,
  COUNT(CASE WHEN interest_map.id IS NULL THEN interest_metrics.interest_id ELSE NULL END) AS not_in_map,
  COUNT(CASE WHEN interest_metrics.interest_id IS NULL THEN interest_map.id ELSE NULL END)  AS not_in_metrics
FROM fresh_segments.interest_metrics
FULL OUTER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id;
```
![image](https://user-images.githubusercontent.com/89623051/143396346-00b5f676-ad59-4789-92f3-c0afd36404c4.png)


### 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table

```sql
WITH cte_id_records AS (
SELECT
  id,
  COUNT(*) AS record_count
FROM fresh_segments.interest_map
GROUP BY id
)
SELECT
  COUNT(record_count) AS id_count,
  record_count
FROM cte_id_records
GROUP BY record_count
```

![image](https://user-images.githubusercontent.com/89623051/143396495-ee39d316-b995-42e6-9604-d6cca9fc9988.png)

### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.

```sql
WITH cte_join AS (
SELECT
  interest_metrics.*,
  interest_map.interest_name,
  interest_map.interest_summary,
  interest_map.created_at,
  interest_map.last_modified
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
)
SELECT * FROM cte_join
WHERE interest_id = 21246;
```

![image](https://user-images.githubusercontent.com/89623051/143396706-b4dd749f-5b65-4adf-aa17-41b99ac55517.png)

### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
WITH cte_join AS (
SELECT
  interest_metrics.*,
  interest_map.interest_name,
  interest_map.interest_summary,
  interest_map.created_at,
  interest_map.last_modified
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
)
SELECT count(*) as prior_records FROM cte_join
WHERE TO_DATE(month_year, 'MM YYYY') < created_at
```

![image](https://user-images.githubusercontent.com/89623051/143396830-1ebea995-8ea2-4c1f-bded-3e3969908fed.png)


Having the beginning of the month may just be a proxy for a summary version of all of our aggregated metrics throughout the month - 
so in this case we need to be wary that the month_year column might well be before our created_at column - but it shouldn’t be from an earlier month.

Let’s confirm this by comparing the truncatated beginning of month for each created_at value with the month_year column again:

```sql
WITH cte_join AS (
SELECT
  interest_metrics.*,
  interest_map.interest_name,
  interest_map.interest_summary,
  interest_map.created_at,
  interest_map.last_modified
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
)
SELECT count(*) as prior_records FROM cte_join
WHERE TO_DATE(month_year, 'MM YYYY')::text < month_year
```

![image](https://user-images.githubusercontent.com/89623051/143396952-75355811-a339-48ab-aa59-b21297ab9c50.png)


## B. Interest Analysis

### 1. Which interests have been present in all month_year dates in our dataset?

```sql
WITH cte_interest_months AS (
SELECT
  interest_id,
  COUNT(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
)
SELECT
  total_months,
  COUNT(DISTINCT interest_id) AS interest_count
FROM cte_interest_months
GROUP BY total_months
ORDER BY total_months DESC;
```

![image](https://user-images.githubusercontent.com/89623051/143449681-5737e23a-8522-4f4b-9da9-71beafcaa41f.png)

### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

```sql
WITH cte_interest_months AS (
SELECT
  interest_id,
  COUNT(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
),
total_months_tb as (SELECT
  total_months,
  COUNT(DISTINCT interest_id) AS interest_count
FROM cte_interest_months
GROUP BY total_months
ORDER BY total_months DESC)

select total_months,
interest_count,
ROUND(
    100 * SUM(interest_count) OVER (ORDER BY total_months desc) /
      (SUM(INTEREST_COUNT) OVER ()),
    2
  ) AS cumulative_percentage
from total_months_tb;
```
![image](https://user-images.githubusercontent.com/89623051/143449814-aed485af-39e2-4986-a238-ca5ce1d1a991.png)


### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

```sql
-- firstly get removed interest_id values with less than 6 months of data
WITH cte_removed_interests AS (
SELECT
  interest_id
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
HAVING COUNT(DISTINCT month_year) < 6
)

SELECT
  COUNT(*) AS removed_rows
FROM fresh_segments.interest_metrics 
WHERE  EXISTS (
  SELECT 1
  FROM cte_removed_interests
  WHERE interest_metrics.interest_id = cte_removed_interests.interest_id
);
```

![image](https://user-images.githubusercontent.com/89623051/143449958-9f2eb5e5-128c-43e4-9f9e-df76cedf2218.png)


### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective. 

```sql
WITH cte_removed_interests AS (
SELECT
  interest_id
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
HAVING COUNT(DISTINCT month_year) < 14
)

SELECT
  COUNT(*) AS removed_rows
FROM fresh_segments.interest_metrics 
WHERE  EXISTS (
  SELECT 1
  FROM cte_removed_interests
  WHERE interest_metrics.interest_id = cte_removed_interests.interest_id
);
```

![image](https://user-images.githubusercontent.com/89623051/143450225-6aa631c4-77d8-4702-aea3-aac729abd451.png)

### 5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?

```sql
WITH cte_ranked_interest AS (
SELECT
  interest_metrics.month_year,
  interest_map.interest_name,
  interest_metrics.composition,
  RANK() OVER (
    PARTITION BY interest_map.interest_name
    ORDER BY composition DESC
  ) AS interest_rank
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
),
cte_top_10 AS (
SELECT
  month_year,
  interest_name,
  composition
FROM cte_ranked_interest
WHERE interest_rank = 1
ORDER BY composition DESC
LIMIT 10
),
cte_bottom_10 AS (
SELECT
  month_year,
  interest_name,
  composition
FROM cte_ranked_interest
WHERE interest_rank = 1
ORDER BY composition
LIMIT 10
),
final_output AS (
  SELECT * FROM cte_top_10
  UNION
  SELECT * FROM cte_bottom_10
)
SELECT * FROM final_output
ORDER BY composition DESC;
```

![image](https://user-images.githubusercontent.com/89623051/143450514-898914f7-1466-443a-b7c9-44936f72ed05.png)


## C. Segment Analysis

### 1. Using the complete dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

```SQL
WITH cte_ranked_interest AS (
SELECT
  interest_metrics.month_year,
  interest_map.interest_name,
  interest_metrics.composition,
  RANK() OVER (
    PARTITION BY interest_map.interest_name
    ORDER BY composition DESC
  ) AS interest_rank
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
),
cte_top_10 AS (
SELECT
  month_year,
  interest_name,
  composition
FROM cte_ranked_interest
WHERE interest_rank = 1
ORDER BY composition DESC
LIMIT 10
),
cte_bottom_10 AS (
SELECT
  month_year,
  interest_name,
  composition
FROM cte_ranked_interest
WHERE interest_rank = 1
ORDER BY composition
LIMIT 10
),
final_output AS (
  SELECT * FROM cte_top_10
  UNION
  SELECT * FROM cte_bottom_10
)
SELECT * FROM final_output
ORDER BY composition DESC;

```

![image](https://user-images.githubusercontent.com/89623051/143551168-e6d0fa18-76a1-4401-8419-0fbc6c194ff1.png)


### 2. Which 5 interests had the lowest average ranking value?

```SQL
SELECT
  interest_map.interest_name,
  ROUND(avg(interest_metrics.ranking), 1) AS average_ranking,
  SUM(1) AS record_count  -- this doesn't look right does it?
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
GROUP BY
  interest_map.interest_name
ORDER BY average_ranking
LIMIT 5;
```

![image](https://user-images.githubusercontent.com/89623051/143551271-8ad8c2cb-b443-4ab9-bb4c-16ac7e18c87f.png)

### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?

```SQL
Which 5 interests had the largest standard deviation in their percentile_ranking value?
WITH BASE AS (
SELECT
  interest_metrics.interest_id,
  interest_map.interest_name,
  ROUND(STDDEV(interest_metrics.percentile_ranking::NUMERIC), 1) AS stddev_pc_ranking,
  MAX(interest_metrics.percentile_ranking) AS max_pc_ranking,
  MIN(interest_metrics.percentile_ranking) AS min_pc_ranking,
  COUNT(*) AS record_count
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
GROUP BY
  interest_metrics.interest_id,
  interest_map.interest_name
ORDER BY stddev_pc_ranking DESC)

SELECT * FROM BASE 
WHERE stddev_pc_ranking IS NOT NULL
LIMIT 5;
```

![image](https://user-images.githubusercontent.com/89623051/143551457-f95b5104-90a0-4fb9-8a12-32f13e165a5e.png)


### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?

```SQL
WITH BASE AS (
SELECT
  interest_map.interest_name,
  interest_metrics.month_year,
  interest_metrics.ranking,
  interest_metrics.percentile_ranking,
  interest_metrics.composition
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
WHERE interest_metrics.month_year IS NOT NULL
AND interest_metrics.interest_id IN (6260, 131, 150, 23, 20764)
ORDER BY
  ARRAY_POSITION(ARRAY[6260, 131, 150, 23, 20764]::INTEGER[], interest_metrics.interest_id),
  interest_metrics.month_year)
  
SELECT 
  interest_name,
  month_year, 
  MAX(percentile_ranking) AS max_pc_ranking,
  MIN(percentile_ranking) AS min_pc_ranking
FROM BASE
GROUP BY interest_name, month_year
  
```

![image](https://user-images.githubusercontent.com/89623051/143551671-33cba61e-5e3f-4836-a61b-2ec121824834.png)


## D. Index Analysis

### The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.



### 1. What is the top 10 interests by the average composition for each month?

```sql
WITH cte_index_composition AS (
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    interest_metrics.composition / interest_metrics.index_value AS index_composition,  -- the key!
    DENSE_RANK() OVER (
      PARTITION BY interest_metrics.month_year
      ORDER BY (interest_metrics.composition / interest_metrics.index_value) DESC
    ) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
    ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL
)
SELECT *
FROM cte_index_composition
WHERE index_rank <= 10
ORDER BY month_year;
```

![image](https://user-images.githubusercontent.com/89623051/143551911-eede1cd2-cd23-4860-83e8-5baeb96c4c1e.png)


### 2. For all of these top 10 interests - which interest appears the most often?


```sql
WITH cte_index_composition AS (
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    interest_metrics.composition / interest_metrics.index_value AS index_composition,  -- the key!
    DENSE_RANK() OVER (
      PARTITION BY interest_metrics.month_year
      ORDER BY (interest_metrics.composition / interest_metrics.index_value) DESC
    ) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
    ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL 
)
SELECT interest_name, count(*) as frequency
FROM cte_index_composition
WHERE index_rank <= 10 
group by interest_name
order by frequency desc
limit 3
```

![image](https://user-images.githubusercontent.com/89623051/143552076-19ad31f5-def0-4e34-b2df-c868dbd8de3a.png)


### 3. What is the average of the average composition for the top 10 interests for each month?

```sql
WITH cte_index_composition AS (
  SELECT
    interest_metrics.month_year,
    interest_map.interest_name,
    interest_metrics.composition / interest_metrics.index_value AS index_composition,  -- the key!
    DENSE_RANK() OVER (
      PARTITION BY interest_metrics.month_year
      ORDER BY (interest_metrics.composition / interest_metrics.index_value) DESC
    ) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map
    ON interest_metrics.interest_id = interest_map.id
  WHERE interest_metrics.month_year IS NOT NULL 
)
SELECT month_year, round(avg(index_composition)::numeric,2) as avg_composition
FROM cte_index_composition
WHERE index_rank <= 10 
group by month_year
order by month_year 
```

![image](https://user-images.githubusercontent.com/89623051/143552260-8fd8614c-d037-4e6d-8d4f-56812c5fc914.png)


### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.

