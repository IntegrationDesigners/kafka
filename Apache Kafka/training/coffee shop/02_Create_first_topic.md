# Exercise 2: The Smart Coffee Bar (Kafka Basics)

In this exercise, we will explore the core concepts of Apache Kafka using a relatable scenario: **a smart coffee bar** where orders stream by in real time.

We are going to create a topic, produce messages (as a customer), and consume messages (as a barista).

---

## What Will You Learn?
* Creating and configuring a **Kafka Topic** (including multiple partitions).
* Sending messages (`Producer`).
* Reading messages (`Consumer`).
* Visually tracking messages via the **Kafdrop UI**.

---

## Step 1: Create a Kafka Topic
We need a channel where all coffee orders will arrive. We'll do this via the web interface.

1. Open the Kafdrop UI in your browser at **`http://localhost:9000`**.
2. Click **New Topic** at the top of the menu.
3. Fill in the following details:
   * **Topic Name:** `coffee-orders`
   * **Partitions:** `3` *(handy for seeing later how Kafka distributes messages across different partitions)*
   * **Replication Factor:** `1`
4. Click **Create**.

*(Optionally, via the terminal, this can also be done with: `podman exec -it kafka kafka-topics --create --topic coffee-orders --bootstrap-server localhost:29092 --partitions 3 --replication-factor 1`)*

---

## Step 2: Start a Consumer (The Barista)
Before placing any orders, we will set up a "barista" that listens to our new topic.

1. Open a **new PowerShell terminal** (alongside your running environment).
2. Run the following command to start the console consumer:
   podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --from-beginning
   
## Step 3: Start a Producer (The Customer)

1. Open a **new PowerShell terminal** (alongside your running environment).
2. Run the following command to start the console producer:
	podman exec -it kafka kafka-console-producer --bootstrap-server localhost:29092 --topic coffee-orders --property "parse.key=true" --property "key.separator=:"
3. You will now see an empty prompt (>). Type an order per line below and press Enter after each line:
4. Add next order:

kassa-1:{"orderId": 101, "customer": "Alice", "coffee": "Cappuccino"}
kassa-1:{"orderId": 102, "customer": "Bob", "coffee": "Espresso"}
kassa-1:{"orderId": 103, "customer": "Charlie", "coffee": "Latte Macchiato"}
kassa-1:{"orderId": 104, "customer": "Diana", "coffee": "Americano"}
	
## Step 4: Check the results in the UI

1. Go to: http://localhost:9000/
2. Open your topic: coffee-orders
3. click on View Messages
4. find the correct partition where the messages are stored 

## Step 5: Test with multiple consumers
1. Open multiple new terminals
3. run in each terminal: podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --from-beginning
2. notice that each terminal gets the same stream of events
