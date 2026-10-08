# Exercise 7:  The National Coffee Chain (Kafka & Flink SQL)

In this extension, we scale up the single coffee bar into a fully-fledged chain with multiple locations spread across different
 Flemish cities. This introduces new challenges regarding data modeling, partitioning, and advanced stream analytics using 
 Apache Flink SQL.
---

## Extension Objectives
* Scaling to Multiple Cities: The coffee bar is no longer a single location, 
	but a chain with shops in various Flemish cities (e.g., Antwerp, Ghent, Bruges, Leuven, Hasselt).
* New Topic & Key Structure:
	- Create a new Kafka topic (e.g., coffee-orders-v2).
	- Update the message keys so they include a clear reference to both the city and 
		the specific register (e.g., ghent-kassa-1, antwerp-kassa-2).
* Variable Infrastructure: Not every city will have the same capacity or number of registers (some cities have 2 registers,
	others 3 or more). Take this into account in your partitioning and consumption strateg
* Ultimate Goal — Flink SQL Analysis: Write a comprehensive Flink SQL query that performs real-time 
	analytics to determine which drink is most popular in which city.

---

## Step 1:  Topic & Producer Modification
Make sure your environment is running (podman compose up -d), including the new jobmanager and taskmanager services.

1. Create the required new topics with enough partitions to handle traffic from the different Flemish cities.
2. Configure your console producer (or code) so that the key fully combines the city and register, following the format city:kassa-id (or city_kassa-id).
3. Generate test data for at least 3 different Flemish cities, each with a different number of active registers.

## Step 2: Consumer Groups & Partitioning in a Chain

1. Adapt your consumer groups (barista-team) to see how Kafka distributes messages across different cities and partitions.
2. MAke sure your barista team only receives orders related to their city
3. Analyze how rebalancing behaves when baristas join or drop out in a multi-city scenario.

## Step 3: Flink SQL: The Ultimate Analytical Goal
Let's calculate how many cups of each coffee type are ordered per 1-minute window.

1. Create a Flink SQL Source Table linked to the new multi-city topic.
2. Define the correct fields (including city, register, drink, and event-time/watermarks).
3. Write a stateful Flink SQL query (including windows, such as TUMBLE) that calculates the most popular drink per city over a specific time period.
4. Ensure the results are written to a Flink Sink Table or made visible directly via the Flink SQL CLI / Dashboard.