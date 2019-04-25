
# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share).

#### Query Project Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership.

- Through this project, you will answer these questions:
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?

_______________________________________________________________________________________________________________


## Assignment 04 - Querying data - Answer Your Project Questions

### Your Project Questions

- Answer at least 4 of the questions you identified last week.
- You can use either BigQuery or the bq command line tool.
- Paste your questions, queries and answers below.

- Question 1:

  Does different type of customers (subscriber_type) tend to have different duration of bike using? Can we have special offer for those customers based on the trip duration?

  * Answer:

  |Row	|subscriber_type|	duration_sec |trips |
  |-----|:-------------:|:------------:|:----:|	 
  |1	  |Customer      	| 3718.785     |136809|	 
  |2	  |Subscriber     |	582.764      |846839|

    - We have two types of customers: the Subscriber (annual or 30-day member); and the Customer (24-hour or 3-day member)
    - Overall, the customers trend to have longer trip duration than subscribers. Especially, the customers average trip duration is around one hour, while the subscribers average trip duration is just around 10 minutes.
    - This tells use the two groups of consumer behaviors:
      - the customers membership is more attractive to tourists or exercisers, who trend to use the bike for long time trip between the places of interest and enjoy the view and fresh air;
      - while the subscribers membership is more attractive to commuters, who usually use bike in the morning and afternoon peak hours, and travel between train stations and office buildings.
    - After we know the consumer behaviors of those two types of users, we can have targeted promotions to them, such as offer promotions to customers with the discounts of opening stations in different cities or near places of interests; or offer bike reservation service of populart stations in peak commute hours with charge of fee.     
    - More detailed statistic analysis is needed to exam those two users thoroughly and provide advise in detail.


  * SQL query:
  ```
  #standardSQL
  SELECT subscriber_type, AVG(duration_sec) as duration_sec, count(subscriber_type) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  GROUP BY subscriber_type
  ```


- Question 2:

  Which end stations have the longest trip durations? So that high frequency of maintenance is recommended for those stations.

  * Answer:

  |Row|	end_station_name	                     |avg_duration_sec|	total_duration_sec |	number_of_trips|
  |---|:--------------------------------------:|:--------------:|	:----------------:|	:-------------:|	 
  |1	|Embarcadero at Sansome	                 | 1626.906       |	75158185	        | 46197          |	 
  |2	|San Francisco Caltrain (Townsend at 4th)| 750.699        |	69074831	        | 92014          |	 
  |3	|Harry Bridges Plaza (Ferry Building)	   | 1086.740       |	54538079	        | 50185          |	 
  |4	|San Francisco Caltrain 2 (330 Townsend) | 655.706        |	38498473	        | 58713	         |
  |5	|Powell Street BART	                     | 1236.575       |	33073435	        | 26746	         |
  |6	|Market at 4th                           | 1224.405       |	32767544	        | 26762          |	 
  |7	|Steuart at Market	                     | 806.813        |	31948190	        | 39598          |	 

    - The season that choose end station to check the average trip duration, instead of start station, is that the maintenance make more sense to pick up bikes sitting in the ending station than tracking the bikes than start from one station.
    - The station of Embarcadero at Sansome become the one that has the longest total duration with a relatively high average duration.
    - Maybe because it is a famous places of interest that attracts tourists who trend to use bike longer as customers than subscribers. But we need further analysis to verify this idea.

  * SQL query:

  ```
  #standardSQL
  SELECT end_station_name, AVG(duration_sec) as avg_duration_sec,
    SUM(duration_sec) as total_duration_sec,
    count(trip_id) as number_of_trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  GROUP BY end_station_name
  ORDER BY total_duration_sec DESC
  ```

- Question 3:

  Which bikes (bike_number) have less frequency of usage? Is that because the bike is worn out after long time usage, or the station that the bike is located is not popular?

  * Answer:

  |Row	|bike_number	|number_of_trips	|min_start_date             	|max_end_date	                |service_duration_day	|frequency|
  |-----|:-----------:|:---------------:|:---------------------------:|:---------------------------:|:-------------------:|:-------:|	 
  |1	  |231	        |167	            |2013-09-13 07:42:00.000 UTC	|2016-08-25 18:18:00.000 UTC	|1077	                |0.155    |	 
  |2	  |244	        |161	            |2013-10-13 12:18:00.000 UTC	|2016-08-19 09:04:00.000 UTC	|1040	                |0.155    |	 
  |3	  |216	        |172	            |2013-09-14 20:06:00.000 UTC	|2016-08-28 15:57:00.000 UTC	|1078	                |0.16	    |
  |4	  |121	        |203	            |2013-09-17 08:27:00.000 UTC	|2016-08-29 23:15:00.000 UTC	|1077                	|0.188   	|
  |5	  |673	        |212	            |2013-08-31 12:20:00.000 UTC	|2016-08-31 15:55:00.000 UTC	|1096	                |0.193 	  |
  |6	  |91	          |212	            |2013-08-29 17:04:00.000 UTC	|2016-08-30 19:51:00.000 UTC	|1097	                |0.193	  |
  |7	  |641	        |211	            |2013-09-02 14:39:00.000 UTC	|2016-08-22 09:43:00.000 UTC	|1084	                |0.195	  |

    - Bike number 231, 244, and 216 have the top 3 lowest frequency of usage, they are all use for around 3 years, which is already a long term usage period for a bike.
    - Further analysis such as looking for the stations distribution of each trip for those bikes, the maintenance data (if accessible), to find out whether it is because the bike is worn out after long time usage, or the station that the bike is located is not popular.

  * SQL query:
  ```
  #standardSQL
  SELECT bike_number, count(trip_id) as number_of_trips,
   min(start_date) as min_start_date ,max(end_date) as max_end_date,  
   TIMESTAMP_DIFF(max(end_date), min(start_date), DAY) as service_duration_day,
   ROUND(count(trip_id)/TIMESTAMP_DIFF(max(end_date), min(start_date),DAY),3) as frequency
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  GROUP BY bike_number
  ORDER BY frequency
  ```


- Question 4:

  For the trip more than one day, are there any special reason behind? Can we satisfied those customers with the demand of overnight bike rent?

  * Answer:

  |Row	|subscriber_type	|duration_of_sec	|trips |
  |-----|:---------------:|:---------------:|:----:|	 
  |1	  |Customer	        |284440.21        |	248  |	 
  |2  	|Subscriber	      |220189.333       |	48   |	  

  |Row	|start_station_name                    	|duration_of_sec	  |trips   |	 
  |-----|:-------------------------------------:|:-----------------:|:------:|
  |1	  |Powell Street BART	                    |182836.941	        |17      |	 
  |2	  |Harry Bridges Plaza (Ferry Building)   |262881.077	        |13	     |
  |3	  |University and Emerson	                |350754.5	          |10	     |
  |4	  |California Ave Caltrain Station	      |177728.9	          |10	     |
  |5	  |San Jose Civic Center	                |288886.4	          |10	     |
  |6	  |Civic Center BART (7th at Market)	    |188539.667	        |9	     |
  |7	  |Rengstorff Avenue / California Street	|244429.778	        |9       |	 

  |Row	|end_station_name                    	     |duration_of_sec	   |trips   |	 
  |-----|:----------------------------------------:|:-----------------:|:------:|
  |1	  |California Ave Caltrain Station           |186088.333	       |12      |	 
  |2	  |Powell Street BART                        |205190.583	       |12	    |
  |3	  |Market at 10th         	                 |222041.0           |12	    |
  |4	  |Palo Alto Caltrain Station	               |254725.4	         |10	    |
  |5	  |Civic Center BART (7th at Market)	       |264627.4	         |10	    |
  |6	  |San Francisco Caltrain (Townsend at 4th)	 |178574.0           |9	      |
  |7	  |SJSU 4th at San Carlos	                   |294320.889	       |9       |	 


    - According to assignment 2, we know there 296 trips that trip duration larger than 1 day.
    - The customers has the most trips that have duration larger than 1 day.
    - The most popular start station is Powell Street BART, which next to the San Francisco Visitor Information Centers, indicating that those users may be the tourists that plan to travel the city for a couple of days. And the top 2 station is Harry Bridges Plaza (Ferry Building) , which is also near a lot of places of interest. While the most popular end station is California Ave Caltrain Station, Powell Street BART, and Market at 10th.
    - So, we can tell the stations around places of interests attracts longer duration users. We can have partner-hotels offers or other over-night travel information displayed on those stations for overnight users.
    - By doing deeper analysis in R with data pulling down, we can have more specific recommendations on users who have demand of trip duration larger than 1 day. 

  * SQL query:

  Check subscriber_type:
  ```
  #standardSQL
  SELECT subscriber_type, AVG(duration_sec) as duration_of_sec, count(subscriber_type) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE duration_sec > 60*60*24
  GROUP BY subscriber_type
  ```
  Check start_station:
  ```
  #standardSQL
  SELECT start_station_name, ROUND(AVG(duration_sec),3) as duration_of_sec, count(subscriber_type) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE duration_sec > 60*60*24
  GROUP BY start_station_name
  ORDER by trips DESC
  ```

  Check end_station:
  ```
  #standardSQL
  SELECT end_station_name, ROUND(AVG(duration_sec),3) as duration_of_sec, count(subscriber_type) as trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE duration_sec > 60*60*24
  GROUP BY end_station_name
  ORDER by trips DESC
  ```
