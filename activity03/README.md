# template-activity-03


# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share).

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership.

- Through this project, you will answer these questions:
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 03 - Querying data from the BigQuery CLI - set up

### What is Google Cloud SDK?
- Read: https://cloud.google.com/sdk/docs/overview

- If you want to go further, https://cloud.google.com/sdk/docs/concepts has lots of good stuff.

### Get Going

- Install Google Cloud SDK: https://cloud.google.com/sdk/docs/

- Try BQ from the command line:

  * General query structure

    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'

### Queries

1. Rerun last week's queries using bq command line tool:
  * Paste your bq queries:

- What's the size of this dataset? (i.e., how many trips)

  ```
  bq query --use_legacy_sql=false '
  > SELECT count(*)
  > FROM `bigquery-public-data.san_francisco.bikeshare_trips` '
  Waiting on bqjob_r5384c5a86d815511_0000016154022067_1 ... (1s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 983648 |
  +--------+
  ```

- What is the earliest start time and latest end time for a trip?

  ```
  bq query --use_legacy_sql=false '
  > SELECT min(start_date), max(end_date)
  > FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  Waiting on bqjob_r7ffad1b0d3111e1f_000001615453e26a_1 ... (0s) Current status: DONE   
  +---------------------+---------------------+
  |         f0_         |         f1_         |
  +---------------------+---------------------+
  | 2013-08-29 09:08:00 | 2016-08-31 23:48:00 |
  +---------------------+---------------------+
  ```

- How many bikes are there?

  ```
  bq query --use_legacy_sql=false '
  > SELECT COUNT(DISTINCT bike_number)
  > FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  Waiting on bqjob_r94b72bcddf1114c_00000161545537ab_1 ... (0s) Current status: DONE   
  +-----+
  | f0_ |
  +-----+
  | 700 |
  +-----+
  ```



2. New Query
  * Paste your SQL query and answer the question in a sentence.

- How many trips are in the morning vs in the afternoon?

  Define 0:00 am - 11:59 am as morning, 12:00 am - 4:59 pm as afternoon. If one trip time includes the time of defined morning or afternoon, we count it as +1 for the total number of morning or afternoon trips.

  Consider the trip durations is possible more than 1 day, we have the following conditions to determine the whether one trio is in the morning or in the afternoon, or both, or neither.

  | Number | Conditions             | Trip in the Morning | Trip in the Afternoon |
  | ------ |:----------------------:|:-------------------:|:---------------------:|
  | 1      | Duration > 1 day       | +1                  | +1                    |  
  | 2      | S < 12                 | +1                  |                       |   
  | 3      | S < 12 and E > 12      |                     | +1                    |   
  | 4      | E < 12                 | +1                  |                       |   
  | 5      | E < 12 and S < 17      |                     | +1                    |   
  | 6      | 12 < S < 17            |                     | +1                    |   
  | 7      | 12 < S < 17 and E < 12 | +1                  |                       |   
  | 8      | 12 < E < 17            |                     | +1                    |   
  | 9      | 12 < E < 17 and S < 12 | +1                  |                       |   

  * S for start_date; E for end_date
  * S < 12 means start_data earlier than TIME 12:00:00

  Then we have SQL query:
  Firstly create a new table with new columns of trips_duration, start_time, and end_time under dataset bike_trips_data as table trips_times:

  ```
  bq ls bike_trips_data
    tableId       Type    Labels   Time Partitioning  
    ---------------- ------- -------- -------------------
    total_bikes      TABLE                               
  ```

  ```
  bq query --use_legacy_sql=false --destination_table=bike_trips_data.trips_times '
  SELECT trip_id, start_date, end_date,
  TIMESTAMP_DIFF(end_date, start_date, MINUTE) as trips_duration,
  CAST (start_date as TIME) as start_time,
  CAST (end_date as TIME) as end_time
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  Waiting on bqjob_r5be299bdadfebf6d_0000016163912921_1 ... (15s) Current status: DONE   
  +---------+---------------------+---------------------+----------------+------------+----------+
  | trip_id |     start_date      |      end_date       | trips_duration | start_time | end_time |
  +---------+---------------------+---------------------+----------------+------------+----------+
  |  585556 | 2014-12-25 12:44:00 | 2014-12-25 17:00:00 |            256 |   12:44:00 | 17:00:00 |
  |  585557 | 2014-12-25 12:45:00 | 2014-12-25 17:01:00 |            256 |   12:45:00 | 17:01:00 |
  |  463710 | 2014-09-22 10:47:00 | 2014-09-22 15:03:00 |            256 |   10:47:00 | 15:03:00 |
  | 1314954 | 2016-08-14 10:48:00 | 2016-08-14 15:04:00 |            256 |   10:48:00 | 15:04:00 |
  |  626517 | 2015-01-31 14:52:00 | 2015-01-31 19:08:00 |            256 |   14:52:00 | 19:08:00 |
  | 1245471 | 2016-06-18 14:55:00 | 2016-06-18 19:11:00 |            256 |   14:55:00 | 19:11:00 |
  |  770172 | 2015-05-17 13:57:00 | 2015-05-17 18:13:00 |            256 |   13:57:00 | 18:13:00 |
  |  532633 | 2014-11-06 11:01:00 | 2014-11-06 15:17:00 |            256 |   11:01:00 | 15:17:00 |
  |  383265 | 2014-07-28 16:01:00 | 2014-07-28 20:17:00 |            256 |   16:01:00 | 20:17:00 |
  +---------+---------------------+---------------------+----------------+------------+----------+
  ```

  ```
  bq ls bike_trips_data
    tableId       Type    Labels   Time Partitioning  
    ---------------- ------- -------- -------------------
    total_bikes      TABLE                                                            
    trips_times      TABLE       
  ```

  #1: Duration > 1 day
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE trips_duration > 1440'
  Waiting on bqjob_r78317064d48d2803_00000161634dccd9_1 ... (0s) Current status: DONE   
  +-----+
  | f0_ |
  +-----+
  | 296 |
  +-----+
  ```
  #2: S < 12
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE start_time < TIME (12, 00, 00)'
  Waiting on bqjob_r1927d6685eb922e6_00000161637381a2_1 ... (0s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 412339 |
  +--------+
  ```

  #3: S < 12 and E > 12
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE start_time < TIME (12, 00, 00) AND end_time > TIME (12, 00, 00)'
  Waiting on bqjob_r2f74b37ff3be2a37_00000161639831ac_1 ... (0s) Current status: DONE   
  +-------+
  |  f0_  |
  +-------+
  | 13280 |
  +-------+

  ```

  #4: E < 12
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE end_time < TIME (12, 00, 00)'                                 
  Waiting on bqjob_r65c70182056dda7c_0000016163997e6d_1 ... (0s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 400144 |
  +--------+
  ```

  #5: E < 12 and S < 17
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE end_time < TIME (12, 00, 00) AND start_time < TIME (17, 00, 00)'
  Waiting on bqjob_r33988d4a750686f0_00000161639cb2c3_1 ... (0s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 398590 |
  +--------+
  ```

  #6: 12 < S < 17
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE start_time > TIME (12, 00, 00) AND start_time < TIME (17, 00, 00)'
  Waiting on bqjob_r798db7bb0197cf7d_00000161639df1bf_1 ... (0s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 264144 |
  +--------+
  ```

  #7: 12 < S < 17 and E < 12
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE start_time > TIME (12, 00, 00) AND start_time < TIME (17, 00, 00) AND end_time < TIME (12, 00, 00)'
  Waiting on bqjob_r61ad1da05829c9fb_00000161639edce3_1 ... (0s) Current status: DONE   
  +-----+
  | f0_ |
  +-----+
  | 317 |
  +-----+
  ```

  #8: 12 < E < 17
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE end_time > TIME (12, 00, 00) AND end_time < TIME (17, 00, 00)'
  Waiting on bqjob_r22bb04bc0c23efe0_00000161639fe86e_1 ... (0s) Current status: DONE   
  +--------+
  |  f0_   |
  +--------+
  | 249917 |
  +--------+
  ```

  #9: 12 < E < 17 and S < 12
  ```
  bq query --use_legacy_sql=false '
  SELECT count(*)
  FROM `silent-card-193103.bike_trips_data.trips_times`
  WHERE end_time > TIME (12, 00, 00) AND end_time < TIME (17, 00, 00) AND start_time < TIME (12, 00, 00)'
  Waiting on bqjob_r14a26c5e991853be_0000016163a0a233_1 ... (0s) Current status: DONE   
  +-------+
  |  f0_  |
  +-------+
  | 12049 |
  +-------+
  ```

  According to the result of the query, we have anew table as below:

  | Number | Conditions             | Trip in the Morning | Trip in the Afternoon | Count  |
  | ------ |:----------------------:|:-------------------:|:---------------------:|:------:|
  | 1      | Duration > 1 day       | +1                  | +1                    | 296    |  
  | 2      | S < 12                 | +1                  |                       | 412339 |  
  | 3      | S < 12 and E > 12      |                     | +1                    | 13280  |        
  | 4      | E < 12                 | +1                  |                       | 400144 |  
  | 5      | E < 12 and S < 17      |                     | +1                    | 398590 |
  | 6      | 12 < S < 17            |                     | +1                    | 264144 |  
  | 7      | 12 < S < 17 and E < 12 | +1                  |                       | 317    |
  | 8      | 12 < E < 17            |                     | +1                    | 249917 |  
  | 9      | 12 < E < 17 and S < 12 | +1                  |                       | 12049  |
  | -      | **Total Count**        | **825145**          | **926227**            | -      |

  Answer: 825145 trips are in the morning, and 926227 trips are in the afternoon.

### Project Questions
- Identify the main questions you'll need to answer to make recommendations (list below, add as many questions as you need). You'll answer these questions in a later assignment.

- Question 1:

  Does different type of customers (subscriber_type) tend to have different duration of bike using? Can we have special offer for those customers based on the trip duration?

- Question 2:

  Which stations have the longest trip durations? So that high frequency of maintenance is recommended for those stations.

- Question 3:

  Which bikes (bike_number) have reducing frequency of usage? Is that because the bike is worn out after long time usage, or the station that the bike is located is not popular?  

- Question 4:

  For the trip more than one day, are there any special reason behind? Can we satisfied those customers with the demand of overnight bike rent?
