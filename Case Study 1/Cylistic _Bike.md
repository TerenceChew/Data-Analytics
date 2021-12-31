<h1> Google Data Analytics Capstone Project </h1>  

Analyst: Terence  
Date: 28.12.21  
Client: Cylistic  


Synopsis
---
This is a capstone project from the Google Data Analytics course from Coursera. Characters in this project are fictional.  
This analysis will be performed in BigQuery's SQL workspace. On the other hand, visualizations will be created with Google Data Studio.

Scenario
---
The company, Cylistic, has launched a bike-share programm. Customers who purchase one-time or full-day pass are referred to as casual riders. Customers who subscribe to annual membership are considered as members.  

Cylistic's finance analysts have concluded that members are more profitable than casual riders. They believe that maximizing the number of members would be the key to future growth.

Instead of organizing marketing campaigns to attract new customers, the company believes that there is a high probabilty to convert casual riders into members as they are already aware of the programm and have choosen it for their mobility needs.

Cylistic's primary objective is to design marketing strategies that focus on converting casual riders into members. In order to do that, the marketing analytics team has to better understand their customers.

The company is interested to utilized historical trip data to idenfity insightful trends.  

1: Ask
---  
<h3> 1.1: Identify the audience, business questions and objectives </h3>

According to the senario given above, the 3 questions that need to be addressed in order to improve Cylistic's future growth are:

1. How do casual riders and members use Cylistic bikes differently?
2. Why would casual riders subscribe to an annual membership?
3. How could the marketing team design strategies to influence casual riders to become members?

And the audience is:
1. Cylistics marketing analytics team
2. Cylistics executive team

2: Prepare
---
<h3> 2.1: Data obtainment </h3>

Data is obtained from Cylistic's history trip data [source](https://divvy-tripdata.s3.amazonaws.com/index.html). The data has been made available by Motivate International Inc. under this [license](https://www.divvybikes.com/data-license-agreement).  

The data used is situated within a time span of 12 months, from November 2020 to October 2021 and is stored on Google Cloud Storage.

<h3> 2.2: Data preparation </h3>

The data is converted from .csv format into tables saved in BigQuery's SQL workspace and is renamed as seen in the queries below.

1. Combine 12 tables into 1 (1 table from each month for a total of 12 months)
2. Only 3 relevant columns are selected and renamed
3. New table created: ori_tripdata_12mths

~~~~sql
SELECT 
    started_at AS start_time, 
    ended_at AS end_time,
    member_casual AS membership_type
FROM 
    `project-1-321206.cyclistic.202011_tripdata` AS first
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202012_tripdata` AS second
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202101_tripdata` AS third
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202102_tripdata` AS fourth
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202103_tripdata` AS fifth
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202104_tripdata` AS sixth
UNION ALL 
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202105_tripdata` AS seventh
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202106_tripdata` AS eighth
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202107_tripdata` AS ninth 
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202108_tripdata` AS tenth
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202109_tripdata` AS eleventh
UNION ALL
SELECT 
    started_at,
    ended_at,
    member_casual
FROM 
    `project-1-321206.cyclistic.202110_tripdata` AS twelfth
~~~~

3: Process
---

<h3> 3.1: Improve data structure </h3>

1. Sort data from old to new
2. Add row numbers for sorting convenience
3. New table created: v1_tripdata

~~~~sql
SELECT 
    row_number() OVER (ORDER BY start_time) AS row_num,
    *
FROM 
    `project-1-321206.cyclistic.ori_tripdata_12mths`
~~~~

1. Determine trip length in minutes, weekday name, weekday name as number (Sunday = 0), month  
2. New table created: v2_tripdata

~~~~sql
SELECT 
    row_num,
    start_time,
    end_time,
    membership_type,
    date_diff(end_time, start_time, MINUTE) AS trip_length_minute,
    format_date("%a", start_time) AS weekday_name,
    format_date("%w", start_time) AS weekday_name_as_num,           # Sunday = 0
    format_date("%b", start_time) AS month
FROM 
    `project-1-321206.cyclistic.v1_tripdata`
ORDER BY 
    row_num
~~~~

<h3> 3.2: Verify data integrity </h3>  

1. Check for errors
2. Identify trip length that are less than 0

~~~~sql
SELECT 
    *
FROM 
    `project-1-321206.cyclistic.v2_tripdata`
WHERE 
    trip_length_minute < 0          
~~~~

The query returned:  

![](https://i.postimg.cc/T1jgfhHy/cylistic-1.png)  

We can easily identify that the negative trip length value is caused by the mispositioning of start time and end time. 

1. Check for NULL values
~~~~sql
SELECT 
    *
FROM 
    `project-1-321206.cyclistic.v2_tripdata`
WHERE 
    trip_length_minute IS NULL
~~~~

The query returned no results. Thus, we are good to proceed with data cleaning.  

<h3> 3.3: Data cleaning </h3>

1. Rectify errors
2. Relocate start time and end time into correct position
3. Remove nagative trip length values

~~~~sql
UPDATE 
    `project-1-321206.cyclistic.v2_tripdata`
SET 
    start_time = end_time,
    end_time = start_time,
    trip_length_minute = abs(trip_length_minute)
WHERE 
    trip_length_minute < 0 
~~~~

<h3> 3.4: Final preparation for analysis </h3>  

1. Last filtering and cleaning of data
2. Ensure data is ready for analysis
3. New table created: v3_tripdata  

~~~~sql
SELECT
    membership_type,
    DATE(start_time) AS date,
    trip_length_minute,
    weekday_name,
    weekday_name_as_num,
    month
FROM 
    `project-1-321206.cyclistic.v2_tripdata`
ORDER BY 
    start_time
~~~~



