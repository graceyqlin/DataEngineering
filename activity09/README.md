# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking:
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze these two
event types.
- This task will be spread out over the last four assignments (9-12).

---

# Assignment 09

## Follow the steps we did in class
- for both the simple flask app and the more complex one.

### Turn in your `/assignment-09-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example.

   - We setup a web server by flask;
   - run a web API service, which services web API calls by writing them to a kafka topic "events";
   - use curl make web API calls to our web service to test;
   - and then manually consume the kafka "events" topic to verify our web service is working.   

2) Your `docker-compose.yml`

3) Source code for the flask application(s) used.

   Use the python flask library to write the simple API server:
   ```
   #!/usr/bin/env python
   from flask import Flask
   app = Flask(__name__)

   @app.route("/")
   def default_response():
       return "This is the default response!"

   @app.route("/purchase_a_sword")
   def purchase_sword():
       # business logic to purchase sword
       return "Sword Purchased!"
   ```

   Edit our python flask script to publish to the kafka topic by KafkaProducer, instead of standard output:
   ```
   #!/usr/bin/env python
   from kafka import KafkaProducer
   from flask import Flask
   app = Flask(__name__)
   event_logger = KafkaProducer(bootstrap_servers='kafka:29092')
   events_topic = 'events'

   @app.route("/")
   def default_response():
       event_logger.send(events_topic, 'default'.encode())
       return "This is the default response!"

   @app.route("/purchase_a_sword")
   def purchase_sword():
       # business logic to purchase sword
       # log event to kafka
       event_logger.send(events_topic, 'purchased_sword'.encode())
       return "Sword Purchased!"
   ```


4) Each important step in the process. For each step, include:
  * The command(s)
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.


  - Create a docker cluster with 3 containers: zookeeper, kafka, and mids

    Create a directory
    ```
    mkdir ~/w205/flask-with-kafka
    cd ~/w205/flask-with-kafka
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

  - Install flask into out mids container, so that we can use the python flask module to write a simple API server:

    Create a file ~/w205/flask-with-kafka/game_api.py with the python code as shown in question 3, using the flash module to implement a simple web service;
    and use flash module to implement a simple web service and print the results to standard output.

    Run our python script in the mids container:    
    ```
    docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
    ```
    Output from our python program here as we make our web API calls:
    ```
     * Serving Flask app "game_api"
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
    ```

  - In a new terminal window, Using a curl to make some web API calls manually; and stop flask with a control-C.

    Code:
    ```
    docker-compose exec mids curl http://localhost:5000/
    ```
    Output in this window:
    ```
    This is the default response!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [14/Mar/2018 01:54:15] "GET / HTTP/1.1" 200 -
    ```

    Code of purchase_a_sword:
    ```
    docker-compose exec mids curl http://localhost:5000/purchase_a_sword
    ```
    Output in this window:
    ```
    Sword Purchased!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [14/Mar/2018 01:54:28] "GET /purchase_a_sword HTTP/1.1" 200 -
    ```

  - Then edit the python script, in order to use flash to write to the kafka topic by KafkaProducer, instead of standard output as the previous step.
    Python code is showing in the question 3.
    Then we run the revised python flask script in the mid container, again.
    ```
    docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka/game_api.py flask run
    ```
    Output from our python program here as we make our web API calls:
    ```
     * Serving Flask app "game_api"
     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
    ```

  - In a new terminal window, make some web API calls manually, by using curl.
    Code:
    ```
    docker-compose exec mids curl http://localhost:5000/
    ```
    Output in this window:
    ```
    This is the default response!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [14/Mar/2018 02:06:50] "GET / HTTP/1.1" 200 -
    ```

    Code for purchase_a_sword:
    ```
    docker-compose exec mids curl http://localhost:5000/purchase_a_sword
    ```
    Output in this window:
    ```
    Sword Purchased!
    ```
    Output in previous window:
    ```
    127.0.0.1 - - [14/Mar/2018 02:06:56] "GET /purchase_a_sword HTTP/1.1" 200 -
    ```

  - Lastly, we consume the kafka topic events
    ```
    docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t events -o beginning -e"
    ```
    Output:
    ```
    default
    purchased_sword
    ```

  - Final clean up, we stop flask with a control-C and tear down the cluster.
    ```
    docker-compose down
    ```
