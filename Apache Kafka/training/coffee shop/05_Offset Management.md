# Exercise 5:  Offset Management and Resetting Offsets

Understand how Kafka tracks consumer progress using committed offsets and learn how to manually reset or shift 
consumer group offsets to replay historical data without losing messages.
---

## What Will You Learn?
* A running Kafka and Kafdrop environment via Podman Compose.
* An existing topic (e.g., coffee-orders) with active partitions (e.g., 3 partitions).
* An active consumer group named barista-team.

---

## Step 1: Produce a New Batch of Test Data
We are going to add a property defining a fixed group name (barista-team)

1. Open Terminal 1 and start Producer: podman exec -it kafka kafka-console-producer --bootstrap-server localhost:29092 --topic coffee-orders --property "parse.key=true" --property "key.separator=:"

kassa-1:{"orderId": 1011, "customer": "Liam", "coffee": "Cappuccino"}
>kassa-2:{"orderId": 1012, "customer": "Olivia", "coffee": "Latte"}
>kassa-3:{"orderId": 1013, "customer": "Noah", "coffee": "Espresso"}
>kassa-1:{"orderId": 1014, "customer": "Emma", "coffee": "Flat White"}
>kassa-2:{"orderId": 1015, "customer": "Lucas", "coffee": "Americano"}
>kassa-3:{"orderId": 1016, "customer": "Mila", "coffee": "Cortado"}

---

## Step 2: Consume and Commit Offsets

1. Start your 3 consumers belonging to the barista-team group
podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders.v1 --group barista-team


## Step 3: Simulate a Replay
We will be sending a series of orders to investigate how the tasks are devided

1. Stop your barista-team (all consuming terminals must be closed)
2. Reset offest: podman exec -it kafka kafka-consumer-groups --bootstrap-server localhost:29092 --group barista-team --topic coffee-orders.v1 --reset-offsets --shift-by -5 --execute
3. Restart your barista team
	podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders.v1 --group barista-team