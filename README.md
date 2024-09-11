# Cyclistic-Project

## Project 1: Google Data Analytics Capstone Project - Cyclistic

Hello there! Welcome to my first project. In this case study, I assume the role of a data analyst working with the marketing analytics team at Cyclistic, a bike-share company in Chicago. 

## Step 1: ASK 

### The business task

Identify how annual members and casual members use Cyclistic bikes differently and provide recommendations to convert casual riders to annual members. 

### Stakeholders

* Lily Moreno - Director of Marketing at Cyclistic.
* Cyclistic's marketing analytics team.
* Cyclistic's executive team. 

## Step 2: Prepare

## Step 3: Process 

**Tool(s): SQL BigQuery**

### 1. Combining 12 months of historical trip data into a single table. 

Note: With a total of 5,743,278 rows - SQL was the preferred choice over spreadsheets. 

### 2. Removing NULL values.

```
{
SELECT
  COUNT(*) - COUNT(ride_id) AS ride_id_null,
  COUNT(*) - COUNT(rideable_type) AS rideable_type_null,
  COUNT(*) - COUNT(started_at) AS started_at_null,
  COUNT(*) - COUNT(ended_at) AS ended_at_null,
  COUNT(*) - COUNT(start_station_name) AS start_station_name_null,
  COUNT(*) - COUNT(start_station_id) AS start_station_id_null,
  COUNT(*) - COUNT(end_station_name) AS end_station_name_null,
  COUNT(*) - COUNT(end_station_id) AS end_station_id_null,
  COUNT(*) - COUNT(start_lat) AS start_lat_null,
  COUNT(*) - COUNT(start_lng) AS start_lng_null,
  COUNT(*) - COUNT(end_lat) AS end_lat_null,
  COUNT(*) - COUNT(end_lng) AS end_lng_null,
  COUNT(*) - COUNT(member_casual) AS member_casual_null
FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data`
}
```

* Columns with NULL values:
* Start_station_name (905,237)
* Start_station_id (905,237)
* End_station_name (956,579)
* End_station_id (956,579)
* End_lat (7,684)
* End_lng (7,684)

```
{
DELETE FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data`
WHERE 
  start_station_name IS NULL OR
  start_station_id IS NULL OR
  end_station_name IS NULL OR
  end_station_id IS NULL OR
  end_lat IS NULL OR
  end_lng IS NULL;
}
```

### 3. Check for duplicates of primary key: ride_id 

```
{
SELECT
  COUNT(ride_id) - COUNT(DISTINCT(ride_id)
FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data` 
}
```

Output: 0 duplicates.

### 4. Split the started_at & ended_at fields into separate date & time columns for easier analysis.
At the same time, remove unnecessary fields for analysis (start_lat, start_lng, end_lat, end_lng)

```
{
CREATE TABLE `capstone-cyclistic-434906.Historical_Data.Combined_Data_v2` AS
(
  SELECT
    ride_id,
    rideable_type,
    CAST(started_at AS DATE) AS started_date,
    CAST(started_at AS TIME) AS started_time,
    start_station_name,
    start_station_id,
    CAST(ended_at AS DATE) AS ended_date,
    CAST(ended_at AS TIME) AS ended_time,
    end_station_name,
    end_station_id,
    member_casual
  FROM FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data`
)
}
```

### 5. Add new columns: ride_length and day_of_week

**Ride Length**
```
{
CREATE TABLE `capstone-cyclistic-434906.Historical_Data.Combined_Data_v3` AS
(
  SELECT
    *,
    CASE
      WHEN TIME_DIFF(ended_time,started_time, MINUTE) >= 0 THEN TIME_DIFF(ended_time,started_time, MINUTE)
      ELSE TIME_DIFF(ended_time,started_time, MINUTE) + 1440
    END
    AS ride_length
  FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data_v2`
)
}
```

**Day of Week**

Firstly, modify all dates to the following format: YYYY-MM-DD. 
```
{
UPDATE `capstone-cyclistic-434906.Historical_Data.Combined_Data_v3`
SET ended_date = CAST(CONCAT("20",RIGHT(CAST(ended_date AS STRING),2),"-",SUBSTR(CAST(ended_date AS STRING),6,2),"-",SUBSTR(CAST(ended_date AS STRING),3,2)) AS DATE)
WHERE ride_id IN
    (
  SELECT
    ride_id
  FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data_v3`
  WHERE
    LEFT(CAST(ended_date AS STRING), 2) = "00"
  )
}
```
Repeat the above for started_date column. 

Next. add the new columns: year_of_ride, month_of_ride, day_of_week.

```
{
CREATE TABLE `capstone-cyclistic-434906.Historical_Data.Combined_Data_v4` AS
(
SELECT
  *,
  EXTRACT(YEAR FROM started_date) AS year_of_ride,
  SUBSTR(CAST(started_date AS STRING),6,2) AS month_of_ride,
  FORMAT_DATE('%A', DATE(started_date)) AS day_of_week
FROM `capstone-cyclistic-434906.Historical_Data.Combined_Data_v3`
)
}
```

### 6. Finally, remove ride_lengths < 1 as these rides are likely to be unintentional, system glitches or that the user changed his/her mind before even riding. 

## Step 4: ANALYZE

**Tool(s): SQL BigQuery + Spreadsheets.**


## Step 5: SHARE 
![Tableau Dashboard](Dashboard 1.png)
