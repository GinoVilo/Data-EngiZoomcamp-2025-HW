# Data Engineering Zoomcamp - Week 2 Homework (Workflow Orchestration)

## Module 2 Homework 

## Solutions

### Question 1
**Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size?**

Options:
- 128.3 MB
- 134.5 MB
- 364.7 MB
- 692.6 MB

**Answer: 134.5 MB**

File size verification:
```bash
$ ls -la yellow_tripdata_2020-12.csv
-rw-r--r-- 1 gv-ia gv-ia 134481400 jul 14 2022 yellow_tripdata_2020-12.csv
Question 2
What is the rendered value of the variable file when:

taxi is set to green
year is set to 2020
month is set to 04

Options:

{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv
green_tripdata_2020-04.csv
green_tripdata_04_2020.csv
green_tripdata_2020.csv

Answer: green_tripdata_2020-04.csv
Question 3
How many rows are there for the Yellow Taxi data for 2020?
Options:

13,537,299
24,648,499
18,324,219
29,430,127

Answer: 24,648,499
Verification Query:
sqlCopySELECT COUNT(*) as total_rows FROM public.yellow_tripdata;

+------------+
| total_rows |
|------------|
| 24648499   |
+------------+
Question 4
How many rows are there for the Green Taxi data for 2020?
Options:

5,327,301
936,199
1,734,051
1,342,034

Answer: 1,734,051
Verification Query:
sqlCopySELECT 
    COUNT(*) as total_rows_2020
FROM 
    public.green_tripdata 
WHERE 
    DATE_TRUNC('year', lpep_pickup_datetime) = '2020-01-01'
    OR filename LIKE 'green_tripdata_2020-__.csv';

+-----------------+
| total_rows_2020 |
|-----------------|
| 1734052         |
+-----------------+
Question 5
How many rows are there for the Yellow Taxi data for March 2021?
Options:

1,428,092
706,911
1,925,152
2,561,031

Answer: 1,925,152
Verification Query:
sqlCopySELECT COUNT(*)
FROM yellow_tripdata
WHERE
    EXTRACT(YEAR FROM tpep_pickup_datetime) = 2021
    AND EXTRACT(MONTH FROM tpep_pickup_datetime) = 3;

+---------+
| count   |
|---------|
| 1925130 |
+---------+
Question 6
How would you configure the timezone to New York in a Schedule trigger?
Options:

Add a timezone property set to EST in the Schedule trigger configuration
Add a timezone property set to America/New_York in the Schedule trigger configuration
Add a timezone property set to UTC-5 in the Schedule trigger configuration
Add a location property set to New_York in the Schedule trigger configuration

Answer: Add a timezone property set to America/New_York in the Schedule trigger configuration
Example:
yamlCopytriggers:
  - id: schedule
    type: schedule
    cron: "0 0 * * *"
    timezone: America/New_York
Technologies Used

Kestra for workflow orchestration
PostgreSQL for data storage
Docker for containerization
pgcli for database queries

Setup Notes
All queries were executed using PostgreSQL 15.10 on Docker with the following connection:
bashCopypgcli -h localhost -p 5432 -U kestra -d postgres-zoomcamp