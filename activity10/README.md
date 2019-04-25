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

# Assignment 10

## Follow the steps we did in class


### Turn in your `/assignment-10-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example.

  - We setup a web server by flask;
  - run a web API service, which services web API calls by writing them to a kafka topic "events";
  - use curl make web API calls to our web service to test;
  - manually consume the kafka "events" topic to verify our web service is working.   
  - and then, consume kafka by using pyspark, the python interface to spark.

2) Your `docker-compose.yml`


3) Source code for the flask application(s) used.

Write the python script that use flask library to publishes to the kafka topic, by KafkaProducer class.
```
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
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

4) Each important step in the process. For each step, include:
  * The command(s)
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.

- Create a docker cluster with 4 containers: zookeeper, kafka, spark, and mids.

  Create a directory
  ```
  mkdir ~/w205/flask-with-kafka-and-spark/
  cd ~/w205/flask-with-kafka-and-spark/
  ```
  Create a docker-compose.yml and run the docker cluster:
  ```
  docker-compose up -d
  ```

- Create a kafka topic called "events":
  ```
  docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
  ```
  the output is:
  ```
  Created topic "events".
  ```

- Install flask into out mids container, so that we can use the python flask module to write a API server:

  Create a file ~/w205/flask-with-kafka-and-spark/game_api_with_json_events.py with the python code as shown in question 3, using the flash module to implement a simple web service;
  and use flash module to implement a simple web service and publish the results to the kafka topic by KafkaProducer.

  Run the python script in the mids container:
  ```
  docker-compose exec mids \
  env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_json_events.py \
  flask run --host 0.0.0.0
  ```

  Output from our python program here as we make our web API calls:
  ```
  * Serving Flask app "game_api_with_json_events"
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
  127.0.0.1 - - [26/Mar/2018 06:04:54] "GET / HTTP/1.1" 200 -
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
  127.0.0.1 - - [26/Mar/2018 06:05:27] "GET /purchase_a_sword HTTP/1.1" 200 -
  ```


- Consume the kafka topic events.
  Run the kafkacat utility in the mids container of our docker cluster to consume the topic:
  ```
  docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
  ```
  Output:
  ```
  {"event_type": "default"}
  {"event_type": "purchase_sword"}
  % Reached end of topic events [0] at offset 2: exiting
  ```

- Then we start to do the activity 2, by continuing with the same cluster and kafka topic we had in activity 1.
  Edit our python flask script game_api_with_json_events.py to run and print output to the command line each time we make a web API call, the new file is named game_api_with_extended_json_events.py. Python code is showing in the question 3.
  Then we run the revised python flask script game_api_with_extended_json_events.py in the mid container, again.
  ```
  docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka-and-spark/game_api_with_extended_json_events.py flask run --host 0.0.0.0
  ```
  Output:
  ```
  * Serving Flask app "game_api_with_extended_json_events"
  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
  ```

- Open a new terminal window, make some web API calls manually, by using curl （Try each command several times in random order）.
  ```
  docker-compose exec mids curl http://localhost:5000/
  ```
  Output in this window:
  ```
  This is the default response!
  ```
  Output in previous window:
  ```
  127.0.0.1 - - [26/Mar/2018 06:40:45] "GET / HTTP/1.1" 200 -
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
  127.0.0.1 - - [26/Mar/2018 06:41:33] "GET /purchase_a_sword HTTP/1.1" 200 -
  ```

- Run the kafkacat utility in the mids container of our docker cluster to consume the topic:
  ```
  docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
  ```
  Output:
  ```
  {"event_type": "default"}
  {"event_type": "purchase_sword"}
  {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  {"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  {"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
  % Reached end of topic events [0] at offset 8: exiting
  ```

- Open a new terminal window and run a pyspark shell.
  ```
  docker-compose exec spark pyspark
  ```
  Output:
  ```
  Welcome to
        ____              __
       / __/__  ___ _____/ /__
      _\ \/ _ \/ _ `/ __/  '_/
     /__ / .__/\_,_/_/ /_/\_\   version 2.2.0
        /_/

  Using Python version 3.6.1 (default, May 11 2017 13:09:58)
  SparkSession available as 'spark'.
  ```

- In the same Spark terminal window, consume the kafka topic events using pyspark.
  ```
  raw_events = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","events").option("startingOffsets", "earliest").option("endingOffsets", "latest").load()
  ```

- Do some manipulations of the events using pyspark:
  create a new data frame with just the value in string format, easy for us to read as human.
  And extract the values into individual json objects.
  ```
  events = raw_events.select(raw_events.value.cast('string'))
  import json
  extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()
  ```


- Check the extracted json values.
  ```
  extracted_events.show()
  ```
  Putput:
  ```
  +--------------+
  |    event_type|
  +--------------+
  |       default|
  |purchase_sword|
  |       default|
  |purchase_sword|
  |       default|
  |       default|
  |purchase_sword|
  |purchase_sword|
  +--------------+
  ```

- Exit pyspark with exit(), and Exit flask with control-C

- Tear down the cluster with:
  ```
  docker-compose down
  ```
