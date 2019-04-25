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

# Assignment 11

## Follow the steps we did in class


### Turn in your `/assignment-11-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example.

  - Created a docker cluster with containers for zookeeper, kafka, spark, and mids.
  - Created a web API server using flask, which has written web logs to a kafka topic in json format, we have read the kafka topic with spark and done some processing.
  - Add cloudera hadoop to the docker cluster and we will write from spark to hdfs as we did in the previous project.
  - Use spark-submit to submit jobs to the spark cluster. We will look at code for submitting spark to other types of clusters, such as standalone, yarn, mesos, and kubernetes. We will look at doing some munging on spark events. We will also look at separating events.

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
from pyspark.sql import SparkSession


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

    events = raw_events.select(raw_events.value.cast('string'))
    extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()

    extracted_events \
        .write \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

Then, we are going to check more spark variations that we can try in pyspark and Jupyter Notebook.
```
#!/usr/bin/env python
"""Extract events from kafka, transform, and write to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf


@udf('string')
def munge_event(event_as_json):
    event = json.loads(event_as_json)
    event['Host'] = "moe" # silly change to show it works
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
    munged_events.show()

    extracted_events = munged_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.munged))) \
        .toDF()
    extracted_events.show()

    extracted_events \
        .write \
        .mode("overwrite") \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

check more spark variations in separating events
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
    event['Host'] = "moe" # silly change to show it works
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

4) Each important step in the process. For each step, include:
  * The command(s)
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.

  - Create a docker cluster with 4 containers: zookeeper, kafka, spark, and mids.

    Create a directory
    ```
    mkdir ~/w205/spark-from-files/
    cd ~/w205/spark-from-files
    ```
    Create a docker-compose.yml
    ```
    cp ~/w205/course-content/11-Storing-Data-III/docker-compose.yml .
    ```
    Copy the python files for the use of future.
    ```
    cp ~/w205/course-content/11-Storing-Data-III/*.py .
    ```

    Then, run the docker cluster:
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

  - Open three new terminal window for cloudera, hadoop, and kafka, respectively, in order to check and watch each of them coming up.
    ```
    docker-compose logs -f cloudera
    docker-compose exec cloudera hadoop fs -ls /tmp/
    docker-compose logs -f kafka
    ```

    The output of cloudera window:
    ```
    cloudera_1   | Start HDFS
    cloudera_1   | starting datanode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-datanode-5ff891e85395.out
    cloudera_1   |  * Started Hadoop datanode (hadoop-hdfs-datanode):
    cloudera_1   | starting namenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-namenode-5ff891e85395.out
    cloudera_1   |  * Started Hadoop namenode:
    cloudera_1   | starting secondarynamenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-secondarynamenode-5ff891e85395.out
    cloudera_1   |  * Started Hadoop secondarynamenode:
    cloudera_1   | Start Components
    cloudera_1   | Press Ctrl+P and Ctrl+Q to background this process.
    cloudera_1   | Use exec command to open a new bash instance for this instance (Eg. "docker exec -i -t CONTAINER_ID bash"). Container ID can be obtained using "docker ps" command.
    cloudera_1   | Start Terminal
    cloudera_1   | Press Ctrl+C to stop instance.
    ```

    The output of hadoop window:
    ```
    Found 2 items
    drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
    drwx-wx-wx   - root   supergroup          0 2018-03-21 01:54 /tmp/hiv
    ```
    The output of kafka window is a very long log list. Showing the last two lines as below:
    ```
    kafka_1      | [2018-03-23 00:46:27,462] DEBUG [Controller id=1] Topics not in preferred replica Map() (kafka.controller.KafkaController)
    kafka_1      | [2018-03-23 00:46:27,462] TRACE [Controller id=1] Leader imbalance ratio for broker 1 is 0.0 (kafka.controller.KafkaController)
    ```

  - Come back to the first terminal window, and create a topic in kafka (same as we have been doing):
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
    docker-compose exec mids env FLASK_APP=/w205/spark-from-files/game_api.py flask run --host 0.0.0.0
    ```
    Output:
    ```
    * Serving Flask app "game_api"
    * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    ```

  - Open a new terminal window, Using a curl to make some web API calls manually; and stop flask with a control-C.

    ```
    docker-compose exec mids curl http://localhost:5000/
    ```
    Output in this window:
    ```
    This is the default response!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [22/Mar/2018 23:30:18] "GET / HTTP/1.1" 200 -
    ```


    ```
    docker-compose exec mids curl http://localhost:5000/purchase_a_sword
    ```
    Output in this window:
    ```
    Sword Purchased!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [22/Mar/2018 23:30:46] "GET /purchase_a_sword HTTP/1.1" 200 -
    ```


  - Consume the kafka topic events.
    Run the kafkacat utility in the mids container of our docker cluster to consume the topic:
    ```
    docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
    ```
    Output:
    ```
    {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
    % Reached end of topic events [0] at offset 8: exiting
    ```

  - Review the following python spark file extract_events.py from the question 3.
    Then, instead of using pyspark we will be using the spark-submit. Submit the extract_events.py file to spark using spark-submit:
    ```
    docker-compose exec spark spark-submit /w205/spark-from-files/extract_events.py
    ```
    Key output:
    ```
    18/03/22 23:36:43 INFO ParquetWriteSupport: Initialized Parquet WriteSupport with Catalyst schema:
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
      } ]
    }
    ```

  - Go to the terminal window with hadoop. Since our code wrote to the hadoop hdfs file system, let's check out and verify this.
    ```
    docker-compose exec cloudera hadoop fs -ls /tmp/
    docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
    ```
    output:
    ```
    Found 2 items
    drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
    drwx-wx-wx   - root   supergroup          0 2018-03-21 01:54 /tmp/hiv
    ```
    ```
    Found 2 items
    -rw-r--r--   1 root supergroup          0 2018-03-22 23:36 /tmp/extracted_events/_SUCCESS
    -rw-r--r--   1 root supergroup       1188 2018-03-22 23:36 /tmp/extracted_events/part-00000-2da18b4f-d840-46be-9c9a-f0fbeaf98777-c000.snappy.parquet
    ```

  - Then, we are going to check more spark variations that we can try in pyspark and Jupyter Notebook.
    ```
    docker-compose exec spark spark-submit /w205/spark-from-files/transform_events.py
    ```
    Key output:
    ```
    18/03/22 23:46:16 INFO CodeGenerator: Code generated in 74.163103 ms
    +--------------------+--------------------+--------------------+
    |                 raw|           timestamp|              munged|
    +--------------------+--------------------+--------------------+
    |{"Host": "localho...|2018-03-22 23:30:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:30:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:30:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:30:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:31:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:31:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:31:...|{"Host": "moe", "...|
    |{"Host": "localho...|2018-03-22 23:31:...|{"Host": "moe", "...|
    +--------------------+--------------------+--------------------+
    ...
    18/03/22 23:46:25 INFO CodeGenerator: Code generated in 93.598422 ms
    +------+-------------+----+-----------+--------------+--------------------+
    |Accept|Cache-Control|Host| User-Agent|    event_type|           timestamp|
    +------+-------------+----+-----------+--------------+--------------------+
    |   */*|     no-cache| moe|curl/7.47.0|       default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|       default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|       default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    |   */*|     no-cache| moe|curl/7.47.0|       default|2018-03-22 23:31:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    +------+-------------+----+-----------+--------------+--------------------+



    ```

  - Then, we are going to check more spark variations in separating events
    ```
    docker-compose exec spark spark-submit /w205/spark-from-files/separate_events.py
    ```
    Key output:
    ```
    18/03/22 23:50:13 INFO CodeGenerator: Code generated in 76.283079 ms
    +------+-------------+----+-----------+--------------+--------------------+
    |Accept|Cache-Control|Host| User-Agent|    event_type|           timestamp|
    +------+-------------+----+-----------+--------------+--------------------+
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    |   */*|     no-cache| moe|curl/7.47.0|purchase_sword|2018-03-22 23:31:...|
    +------+-------------+----+-----------+--------------+--------------------+
    ...
    18/03/22 23:50:14 INFO DAGScheduler: Job 2 finished: showString at NativeMethodAccessorImpl.java:0, took 0.584429 s
    +------+-------------+----+-----------+----------+--------------------+
    |Accept|Cache-Control|Host| User-Agent|event_type|           timestamp|
    +------+-------------+----+-----------+----------+--------------------+
    |   */*|     no-cache| moe|curl/7.47.0|   default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|   default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|   default|2018-03-22 23:30:...|
    |   */*|     no-cache| moe|curl/7.47.0|   default|2018-03-22 23:31:...|
    +------+-------------+----+-----------+----------+--------------------+

    ```

  - Run the jupyter notebook:
    ```
    docker-compose exec spark env
    ```
    Output:
    ```
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
    http://0.0.0.0:8888/?token=b6c6b56f8b1b5282fa651bc82659e86dedd668dccd24b89f
    ```
    copy the url to browser and change the host to science t open the jupyter notebook.


  - Tear down the cluster.
    ```
    docker-compose down
    ```
