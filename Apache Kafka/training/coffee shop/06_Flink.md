# Exercise 6:  Real-Time Coffee Analytics with Apache Flink SQL

Learn how to use Apache Flink to perform stateful stream processing and real-time aggregations (Tumbling Windows) 
directly on top of your existing Kafka coffee-orders topic, without writing complex Java/Scala boilerplate code.
---

## What Will You Learn?
* Connecting Flink SQL to an existing Kafka topic
* Defining event-time and Watermarks in Flink.
* Running stateful stream aggregations (counting orders per coffee type per minute)
* nspecting Flink jobs via the Flink Dashboard UI

---

## Step 1: Open the Flink SQL Client
Make sure your environment is running (podman compose up -d), including the new jobmanager and taskmanager services.

1. Open a terminal and enter the running Flink JobManager container to start the Flink SQL CLI:
	podman exec -it jobmanager ./bin/sql-client.sh
2. You will be greeted by the Flink SQL interactive command line interface.
3. Install required libraries:
podman exec -it jobmanager wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.1.0-1.18/flink-sql-connector-kafka-3.1.0-1.18.jar
podman exec -it taskmanager wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.1.0-1.18/flink-sql-connector-kafka-3.1.0-1.18.jar## Step 2: Create the Kafka Source Table in Flink

Because Flink needs to know the structure of your JSON coffee orders, we define a dynamic Table mapped to your existing Kafka topic coffee-orders.

1. Run the following SQL statement in the Flink SQL CLI:

CREATE TABLE coffee_orders (
    kassa_id STRING,
    orderId BIGINT,
    customer STRING,
    coffee STRING,
    -- We define a processing or event time attribute for windowing
    proctime AS PROCTIME()
) WITH (
    'connector' = 'kafka',
    'topic' = 'coffee-orders',
    'properties.bootstrap.servers' = 'kafka:29092',
    'properties.group.id' = 'flink-analytics-group',
    'scan.startup.mode' = 'earliest-offset',
    'format' = 'json'
);

## Step 3: Run a Real-Time Tumbling Window Aggregation
Let's calculate how many cups of each coffee type are ordered per 1-minute window.

1. Run this query in the Flink SQL CLI:

SELECT 
    coffee,
    COUNT(orderId) AS total_orders,
    window_start,
    window_end
FROM TABLE(
    TUMBLE(TABLE coffee_orders, DESCRIPTOR(proctime), INTERVAL '1' MINUTE)
)
GROUP BY window_start, window_end, coffee;

Now you should see analitics comming in

2. Create new orders

kassa-1:{"orderId": 1201, "customer": "Niki", "coffee": "Cappuccino"}
kassa-2:{"orderId": 1202, "customer": "Sam", "coffee": "Espresso"}
kassa-1:{"orderId": 1203, "customer": "Alex", "coffee": "Cappuccino"}

Wait a minute or two and see the new analytics comming in

## Step 4: go to Flink Dashboard and analyse what you can find

http://localhost:8081/#/overview

note that we did a select statement, so the data is not produced to a topic and so not reproducable.

## Step 5: lets create a sink table

1. Stop the SQL query table
2. Create a sink table defining the output of our flink job

CREATE TABLE coffee_stats_output (
    coffee STRING,
    total_orders BIGINT,
    window_start TIMESTAMP(3),
    window_end TIMESTAMP(3)
) WITH (
    'connector' = 'kafka',
    'topic' = 'coffee-stats-output',
    'properties.bootstrap.servers' = 'kafka:29092',
    'format' = 'json'
);

3. Start the streaming job

INSERT INTO coffee_stats_output
SELECT 
    coffee,
    COUNT(orderId) AS total_orders,
    window_start,
    window_end
FROM TABLE(
    TUMBLE(TABLE coffee_orders, DESCRIPTOR(proctime), INTERVAL '1' MINUTE)
)
GROUP BY window_start, window_end, coffee;

4. now you won't see it live in your terminal but you can see it in your Kafdrop UI
5. You can also create a now consumer of group analytics-team to receive the data