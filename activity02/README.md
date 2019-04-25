# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share).

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership.

- Through this project, you will answer these questions:
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 02: Querying Data with BigQuery

### What is Google Cloud?
- Read: https://cloud.google.com/docs/overview/

### Get Going

- Go to https://cloud.google.com/bigquery/
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, so go ahead (OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO)
- Now you will see the console screen. This is where you can manage everything for GCP
- Go to the menus on the left and scroll down to BigQuery
- Now go to https://cloud.google.com/bigquery/public-data/bay-bike-share
- Scroll down to "Go to Bay Area Bike Share Trips Dataset" (This will open a BQ working page.)


### Some initial queries
Paste your SQL query and answer the question in a sentence.

- What's the size of this dataset? (i.e., how many trips)
  * Answer:  

    | Question           | Answer Number of Trips|
    | -------------------|:---------------------:|
    |Size of this dataset| 983648                |

  * SQL query:
    ```standardSQL
    #standardSQL
    SELECT count(*)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    ```


- What is the earliest start time and latest end time for a trip?
  * Answer:  

    | Question            |Answer Time                  |
    | --------------------|:---------------------------:|
    | Earliest start time | 2013-08-29 09:08:00.000 UTC |
    | Latest end time     | 2016-08-31 23:48:00.000 UTC |

  * SQL query:
    ```standardSQL
    #standardSQL
    SELECT min(start_date), max(end_date)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    ```

- How many bikes are there?
  * Answer:

    | Question           | Answer Number of Bikes|
    | -------------------|:---------------------:|
    | Number of bikes    | 700                   |
    
  * SQL query:

    ```standardSQL
    #standardSQL
    SELECT COUNT(DISTINCT bike_number)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    ```

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- Question 1: How many subscribers trips and customers trips respectively?
  * Answer:

    | Question     |Answer Number of Trips   |
    | -------------|:-----------------------:|
    | Subscriber   | 846839                  |
    | Customer     | 136809                  |

  * SQL query:
    ```standardSQL
    #standardSQL
    SELECT subscriber_type, count(*)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    GROUP BY subscriber_type
    ```

- Question 2:How many trips have duration equal to or less than 1 minute and how many trips have duration larger than 1440 minutes (1 day)?
  * Answer:

    | Question               |Answer Number of Trips |
    | -----------------------|:---------------------:|
    | Duration <= 1 minute   | 2919                  |
    | Duration > 1 day       | 296                   |

  * SQL query:

    To create the table that store the duration as trips_duration:
    ```standardSQL
    #standardSQL
    SELECT trip_id, start_date, end_date,
    TIMESTAMP_DIFF(end_date, start_date, MINUTE) as trips_duration
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    ORDER BY trips_duration DESC
    ```

    Get the number of trips that have duration equal to or less than 1 minute:
    ```standardSQL
    #standardSQL
    SELECT distinct count(*)
    FROM `silent-card-193103.bike_trips_data.trips_duration`
    WHERE trips_duration <= 1
    ```

    Get the number of trips that have duration larger than 1 day:
    ```standardSQL
    #standardSQL
    SELECT distinct count(*)
    FROM `silent-card-193103.bike_trips_data.trips_duration`
    WHERE trips_duration > 1440
    ```


- Question 3: Which station is the most popular one as a start station, and which is the most popular one as the end station? And how many trips are they have respectively?

  * Answer:

    | Question                   | Answer Station Name                      | Number of Trips |
    | ---------------------------|:----------------------------------------:| ---------------:|
    | Most popular start station | San Francisco Caltrain (Townsend at 4th) | 72683           |
    | Most popular end station   | San Francisco Caltrain (Townsend at 4th) |	92014           |

  * SQL query:

    Calculate the number of trips by start_station_name in descending order:
    ```standardSQL
    #standardSQL
    SELECT start_station_name, count(*) as count_trips
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    GROUP BY start_station_name
    ORDER BY count_trips DESC
    ```

    Calculate the number of trips by end_station_name in descending order:
    ```standardSQL
    #standardSQL
    SELECT end_station_name, count(*) as count_trips
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    GROUP BY end_station_name
    ORDER BY count_trips DESC
    ```
