# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking:
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze event types.
- This task will be spread out over the last four assignments (9-12).

## Project 3 Task Options

- All: Game shopping cart data used for homework
- Advanced option 1: Generate and filter more types of items.
- Advanced option 2: Enhance the API to accept parameters for purchases (sword/item type) and filter
- Advanced option 3: Shopping cart data & track state (e.g., user's inventory) and filter


---

# Assignment 12

## Do the following:
- Spin up the cluster with the necessary containers.
- Run your web app.
- Generate data.
- Write events & check to make sure you were successful.
- Run queries with Presto - at least Select * from <your-table-name>

### Turn in your `/assignment-12-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example.

  Adding to the previous Assignment 11, in Assignment 12, we will:

  - Use kafkacat in an interactive mode where it will show events as they come through.
  - Use Apache Bench (now added to midsw205/base) to automate stress testing of our web API. We will add some code to spark to process events with different schemas.
  - Add a new event type to our flask app.
  - Use the overwrite option when write out from spark to automatically overwrite it if the directory already exists.
  - Write spark data frames out and read spark data frames back in.
  - Query using SQL to get the results of a query into Pandas, and use the more convenient Pandas functions to process our results.

2) Your `docker-compose.yml`

3) Source code for the application(s) used.

Edit our python flask script to run and print output to the command line each time we make a web API call:
```
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask, request

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    event.update(request.headers)
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "This is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "Sword Purchased!\n"
```

Review the following python spark file extract_events.py. Instead of using pyspark we will be using the spark-submit.
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe"
    event['Cache-Control'] = "no-cache"
    return json.dumps(event)


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    munged_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .withColumn('munged', munge_event('raw'))

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()

    sword_purchases = extracted_events \
        .filter(extracted_events.event_type == 'purchase_sword')
    sword_purchases.show()
    # sword_purchases \
        # .write \
        # .mode("overwrite") \
        # .parquet("/tmp/sword_purchases")

    default_hits = extracted_events \
        .filter(extracted_events.event_type == 'default')
    default_hits.show()
    # default_hits \
        # .write \
        # .mode("overwrite") \
        # .parquet("/tmp/default_hits")


if __name__ == "__main__":
    main()
```

Change previous code to handle multiple schemas for events.
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))

    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()


if __name__ == "__main__":
    main()
```

Stop the flask web API server to add a new event type of purchase_a_knife.
```
@app.route("/purchase_a_knife")
def purchase_a_knife():
    purchase_knife_event = {'event_type': 'purchase_knife',
                            'description': 'very sharp knife'}
    log_to_kafka('events', purchase_knife_event)
    return "Knife Purchased!\n"
```






4) Each important step in the process. For each step, include:

  - Create a docker cluster with 4 containers: zookeeper, kafka, spark, and mids.

    - Create a directory
      ```
      mkdir ~/w205/full-stack/
      cd ~/w205/full-stack
      ```
    - Create a docker-compose.yml
      ```
      cp ~/w205/course-content/12-Querying-Data-II/docker-compose.yml .
      ```
    - Copy the python files for the use of future, and check it.
      ```
      cp ~/w205/course-content/12-Querying-Data-II/*.py .
      ls -l
      ```
      Output:
      ```
      total 16
      -rw-r--r-- 1 science science 1378 Apr  4 01:44 docker-compose.yml
      -rw-r--r-- 1 science science 1345 Apr  4 01:44 filtered_writes.py
      -rw-r--r-- 1 science science  675 Apr  4 01:44 game_api.py
      -rw-r--r-- 1 science science 1231 Apr  4 01:44 just_filtering.py
      ```

    - Review the docker compose file, vi the file, and change the directory mounts.
      ```
      vi docker-compose.yml
      ```

    - Then, run the docker cluster:
      ```
      docker-compose up -d
      ```
      Output:
      ```
      Creating network "sparkfromfiles_default" with the default driver
      Creating sparkfromfiles_cloudera_1
      Creating sparkfromfiles_zookeeper_1
      Creating sparkfromfiles_mids_1
      Creating sparkfromfiles_spark_1
      Creating sparkfromfiles_kafka_1
      ```

  - Create a topic in kafka:
    ```
    docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
    ```
    Output:
    ```
    Created topic "events".
    ```

  - Install flask into out mids container, so that we can use the python flask module to write a API server:

    Review the copied game_api.py with the python code as shown in question 3, using the flash module to implement a simple web service;
    and use flash module to implement a simple web service and publish the results to the kafka topic by KafkaProducer.

    Run flask with our game_api.py python code in the mids container.
    ```
    docker-compose exec mids env FLASK_APP=/w205/full-stack/game_api.py flask run --host 0.0.0.0
    ```
    Output:
    ```
    * Serving Flask app "game_api"
    * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    ```

  - Open the second terminal windonw. Run kafkacat in continuous mode in a separate window, in order to see events as they come through.
    ```
    docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
    ```
    Output:
    ```
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    {"Host": "user1.comcast.com", "event_type": "default", "Accept": "*/*", "User-Agent": "ApacheBench/2.3"}
    packet_write_wait: Connection to 165.227.0.11 port 22: Broken pipe
    ```

  - Open the third terminal window. Use apache bench as shown below to generate multiple requests of the same thing. The -n option is used below to specify 10 of each.
    ```
    docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
    ```
    Output:
    ```
    This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking localhost (be patient).....done


    Server Software:        Werkzeug/0.14.1
    Server Hostname:        localhost
    Server Port:            5000

    Document Path:          /
    Document Length:        30 bytes

    Concurrency Level:      1
    Time taken for tests:   0.129 seconds
    Complete requests:      10
    Failed requests:        0
    Total transferred:      1850 bytes
    HTML transferred:       300 bytes
    Requests per second:    77.41 [#/sec] (mean)
    Time per request:       12.918 [ms] (mean)
    Time per request:       12.918 [ms] (mean, across all concurrent requests)
    Transfer rate:          13.99 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.1      0       0
    Processing:     3   13   8.8     10      30
    Waiting:        2   11   8.0      9      24
    Total:          3   13   8.8     10      30

    Percentage of the requests served within a certain time (ms)
      50%     10
      66%     17
      75%     20
      80%     21
      90%     30
      95%     30
      98%     30
      99%     30
     100%     30 (longest request)
     ```

     Code:
     ```
     docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
     ```
     Output:
     ```
     This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
     Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
     Licensed to The Apache Software Foundation, http://www.apache.org/

     Benchmarking localhost (be patient).....done


     Server Software:        Werkzeug/0.14.1
     Server Hostname:        localhost
     Server Port:            5000

     Document Path:          /purchase_a_sword
     Document Length:        17 bytes

     Concurrency Level:      1
     Time taken for tests:   0.103 seconds
     Complete requests:      10
     Failed requests:        0
     Total transferred:      1720 bytes
     HTML transferred:       170 bytes
     Requests per second:    97.06 [#/sec] (mean)
     Time per request:       10.303 [ms] (mean)
     Time per request:       10.303 [ms] (mean, across all concurrent requests)
     Transfer rate:          16.30 [Kbytes/sec] received  

     Connection Times (ms)
                    min  mean[+/-sd] median   max
     Connect:        0    0   0.0      0       0
     Processing:     6   10   3.4     10      16
     Waiting:        3    9   4.2      9      16
     Total:          6   10   3.4     10      16

     Percentage of the requests served within a certain time (ms)
        50%     10
        66%     10
        75%     12
        80%     15
        90%     16
        95%     16
        98%     16
        99%     16
        100%     16 (longest request)
     ```

     code:
     ```
     docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
     ```
     Output:
     ```
     This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
     Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
     Licensed to The Apache Software Foundation, http://www.apache.org/

     Benchmarking localhost (be patient).....done


     Server Software:        Werkzeug/0.14.1
     Server Hostname:        localhost
     Server Port:            5000

     Document Path:          /
     Document Length:        30 bytes

     Concurrency Level:      1
     Time taken for tests:   0.075 seconds
     Complete requests:      10
     Failed requests:        0
     Total transferred:      1850 bytes
     HTML transferred:       300 bytes
     Requests per second:    132.53 [#/sec] (mean)
     Time per request:       7.546 [ms] (mean)
     Time per request:       7.546 [ms] (mean, across all concurrent requests)
     Transfer rate:          23.94 [Kbytes/sec] received

     Connection Times (ms)
                   min  mean[+/-sd] median   max
     Connect:        0    0   0.2      0       1
     Processing:     3    7   6.0      7      23
     Waiting:        2    6   6.1      4      22
     Total:          3    7   6.1      7      23

     Percentage of the requests served within a certain time (ms)
      50%      7
      66%      9
      75%      9
      80%      9
      90%     23
      95%     23
      98%     23
      99%     23
      100%     23 (longest request)
     ```

     Code:
     ```
     docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
     ```
     Output:
     ```
     This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
     Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
     Licensed to The Apache Software Foundation, http://www.apache.org/

     Benchmarking localhost (be patient).....done


     Server Software:        Werkzeug/0.14.1
     Server Hostname:        localhost
     Server Port:            5000

     Document Path:          /purchase_a_sword
     Document Length:        17 bytes

     Concurrency Level:      1
     Time taken for tests:   0.105 seconds
     Complete requests:      10
     Failed requests:        0
     Total transferred:      1720 bytes
     HTML transferred:       170 bytes
     Requests per second:    95.53 [#/sec] (mean)
     Time per request:       10.468 [ms] (mean)
     Time per request:       10.468 [ms] (mean, across all concurrent requests)
     Transfer rate:          16.05 [Kbytes/sec] received

     Connection Times (ms)
                   min  mean[+/-sd] median   max
     Connect:        0    1   1.5      0       5
     Processing:     2   10  10.7      4      35
     Waiting:        0    8  10.8      4      34
     Total:          3   10  12.0      5      40

     Percentage of the requests served within a certain time (ms)
      50%      5
      66%      6
      75%     13
      80%     21
      90%     40
      95%     40
      98%     40
      99%     40
      100%     40 (longest request)
     ```


  - Review the following python spark file extract_events.py from the question 3.
    Then, instead of using pyspark we will be using the spark-submit. Submit the extract_events.py file to spark using spark-submit:
    ```
    docker-compose exec spark spark-submit /w205/spark-from-files/separate_events.py
    ```
    Key output:
    ```
    18/04/04 02:06:43 INFO DAGScheduler: Job 0 finished: runJob at PythonRDD.scala:446, took 3.950228 s
    root
     |-- Accept: string (nullable = true)
     |-- Host: string (nullable = true)
     |-- User-Agent: string (nullable = true)
     |-- event_type: string (nullable = true)
     |-- timestamp: string (nullable = true)
    ...
    18/04/04 02:06:45 INFO CodeGenerator: Code generated in 147.995745 ms
    +------+-----------------+---------------+--------------+--------------------+
    |Accept|             Host|     User-Agent|    event_type|           timestamp|
    +------+-----------------+---------------+--------------+--------------------+
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    +------+-----------------+---------------+--------------+--------------------+
    ```

  - Use the code of just_filtering.py from question 3, to use spark-submit and handle multiple schemas for events.
    ```
    docker-compose exec spark spark-submit /w205/full-stack/just_filtering.py
    ```
    Key Output:
    ```
    18/04/04 02:22:16 INFO DAGScheduler: Job 0 finished: runJob at PythonRDD.scala:446, took 3.812941 s
    root
     |-- Accept: string (nullable = true)
     |-- Host: string (nullable = true)
     |-- User-Agent: string (nullable = true)
     |-- event_type: string (nullable = true)
     |-- timestamp: string (nullable = true)
     ...
     18/04/04 02:22:17 INFO CodeGenerator: Code generated in 37.10852 ms
    +------+-----------------+---------------+--------------+--------------------+
    |Accept|             Host|     User-Agent|    event_type|           timestamp|
    +------+-----------------+---------------+--------------+--------------------+
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    +------+-----------------+---------------+--------------+--------------------+
    ...
    18/04/04 02:22:21 INFO ParquetWriteSupport: Initialized Parquet WriteSupport with Catalyst schema:
    {
      "type" : "struct",
      "fields" : [ {
        "name" : "Accept",
        "type" : "string",
        "nullable" : true,
        "metadata" : { }
      }, {
        "name" : "Host",
        "type" : "string",
        "nullable" : true,
        "metadata" : { }
      }, {
        "name" : "User-Agent",
        "type" : "string",
        "nullable" : true,
        "metadata" : { }
      }, {
        "name" : "event_type",
        "type" : "string",
        "nullable" : true,
        "metadata" : { }
      }, {
        "name" : "timestamp",
        "type" : "string",
        "nullable" : true,
        "metadata" : { }
      } ]
    }
    ```

  - Check our hadoop hdfs to make sure it's there.
    code:
    ```
    docker-compose exec cloudera hadoop fs -ls /tmp/
    ```
    Output:
    ```
    Found 3 items
    drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
    drwx-wx-wx   - root   supergroup          0 2018-04-04 01:47 /tmp/hive
    drwxr-xr-x   - root   supergroup          0 2018-04-04 02:22 /tmp/purchases
    ```

    Code:
    ```
    docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
    ```
    Output:
    ```
    Found 2 items
    -rw-r--r--   1 root supergroup          0 2018-04-04 02:22 /tmp/purchases/_SUCCESS
    -rw-r--r--   1 root supergroup       1653 2018-04-04 02:22 /tmp/purchases/part-00000-97ad78f3-d9ce-4c62-ba8b-d5eb3347c39e-c000.snappy.parquet
    ```

  - Startup a jupyter notebook. Remember that to access it from our laptop web browser, we will need to change the IP address to the IP address of our droplet.
    ```
    docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
    ```
    Output:
    ```
    [I 02:26:48.861 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
    [C 02:26:48.862 NotebookApp]

        Copy/paste this URL into your browser when you connect for the first time,
        to login with a token:
            http://0.0.0.0:8888/?token=d15a9095f27ec2e5845a4a2222aca0d92da041b0d8f2c554
    ```

  - Copy the URL to brower and change the host notebook0.0.0.0 to to science to open the jupyter.
    In the jupyter notebook, run each of the following in a separate cell.
    Code:
    ```
    purchases = spark.read.parquet('/tmp/purchases')
    ```

    Code:
    ```
    purchases.show()
    ```
    Output:
    ```
    +------+-----------------+---------------+--------------+--------------------+
    |Accept|             Host|     User-Agent|    event_type|           timestamp|
    +------+-----------------+---------------+--------------+--------------------+
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    |   */*|    user2.att.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:57:...|
    +------+-----------------+---------------+--------------+--------------------+
    ```

    Register as table. Becasue it is easy to writing SQL than transformed code.
    Code:
    ```
    purchases.registerTempTable('purchases')
    ```


    Use SQL to do query. Code:
    ```
    purchases_by_example2 = spark.sql("select * from purchases where Host = 'user1.comcast.com'")
    ```


    Code:
    ```
    purchases_by_example2.show()
    ```
    Output:
    ```
    +------+-----------------+---------------+--------------+--------------------+
    |Accept|             Host|     User-Agent|    event_type|           timestamp|
    +------+-----------------+---------------+--------------+--------------------+
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    |   */*|user1.comcast.com|ApacheBench/2.3|purchase_sword|2018-04-04 01:56:...|
    +------+-----------------+---------------+--------------+--------------------+
    ```

    #make parallel data frame, in to pandas. Because pandas is small. but lose the parallel. Code:
    ```
    df = purchases_by_example2.toPandas()
    ```


    Code:
    ```
    df.describe()
    ```
    Output (in Pandas format):

    | Accept	| Host	| User-Agent	| event_type	| timestamp |
    |:-------:|:------:|:----------:|:------------:|:---------:|
    |count	| 10	| 10	| 10	| 10	| 10 |
    | unique	| 1	| 1	| 1	| 1	| 10 |
    |top	| */*	| user1.comcast.com	| ApacheBench/2.3	| purchase_sword	| 2018-04-04 | 01:56:50.198 |
    | freq	| 10	| 10	| 10	| 10	| 1 |
    


  - Tear down the cluster.
    ```
    docker-compose down
    ```
