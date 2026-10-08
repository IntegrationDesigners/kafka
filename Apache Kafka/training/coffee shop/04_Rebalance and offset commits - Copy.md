# Exercise 4:  "The Outage" — Consumer Rebalance & Offset Commits

In a real coffee shop, a barista might suddenly fall ill, or a server might restart. 
In this exercise, we discover how Kafka automatically redistributes tasks (Rebalancing) and how Kafka keeps track of what has already been processed (Offsets).

---

## What Will You Learn?
* Consumer Rebalance: What Kafka does automatically when a consumer (barista) suddenly drops out or joins.
* Offset Committing: How a Consumer Group precisely remembers which message was read last, ensuring you don't lose data upon a restart.


---

## Step 1: Start Two Baristas (with Key Display)
We are going to add a property defining a fixed group name (barista-team)

1. Open Terminal 1 and start Barista 1: podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team
2. Open Terminal 2 and start Barista 2: podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team 
3. Open Terminal 3 and start Barista 3: podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team 

---

## Step 2: Send an Initial Batch of Orders

1. Open a third terminal (producer) and send in a small batch of orders: podman exec -it kafka kafka-console-producer --bootstrap-server localhost:29092 --topic coffee-orders --property "parse.key=true" --property "key.separator=:"

kassa-1:{"orderId": 701, "customer": "Sophie", "coffee": "Caramel Macchiato"}
>kassa-2:{"orderId": 702, "customer": "Daan", "coffee": "Americano"}
>kassa-3:{"orderId": 703, "customer": "Lotte", "coffee": "Flat White"}
>kassa-1:{"orderId": 704, "customer": "Finn", "coffee": "Espresso"}
>kassa-2:{"orderId": 705, "customer": "Floor", "coffee": "Cappuccino"}
>kassa-3:{"orderId": 706, "customer": "Milan", "coffee": "Latte"}
>kassa-1:{"orderId": 707, "customer": "Tessa", "coffee": "Cortado"}
>kassa-2:{"orderId": 708, "customer": "Jasper", "coffee": "Mocha"}
>kassa-3:{"orderId": 709, "customer": "Evi", "coffee": "Ristretto"}
>kassa-1:{"orderId": 710, "customer": "Stijn", "coffee": "Cold Brew"}
## Step 3: Simulate an Outage (Crash)
We will be sending a series of orders to investigate how the tasks are devided

1. Suppose Barista 2 (Terminal 2) suddenly drops out (for example, due to a crash or closing the window).
2. Go to Terminal 2 (Barista 2) and abruptly close the window (or press Ctrl+C).
3. What happens in Kafdrop? If you check http://localhost:9000 under the consumer group barista-team, you will see that the number of active members drops from 2 to 1. Kafka notices this within seconds.
4. The Rebalance: Barista 1 (Terminal 1) automatically gets assigned the partitions of the fallen colleague.
	
## Step 4: Send New Orders During the Outage
Send a few new orders via your producer terminal while Barista 2 is offline:

kassa-1:{"orderId": 801, "customer": "Sophie", "coffee": "Caramel Macchiato"}
>kassa-2:{"orderId": 802, "customer": "Daan", "coffee": "Americano"}
>kassa-3:{"orderId": 803, "customer": "Lotte", "coffee": "Flat White"}
>kassa-1:{"orderId": 804, "customer": "Finn", "coffee": "Espresso"}
>kassa-2:{"orderId": 805, "customer": "Floor", "coffee": "Cappuccino"}
>kassa-3:{"orderId": 806, "customer": "Milan", "coffee": "Latte"}
>kassa-1:{"orderId": 807, "customer": "Tessa", "coffee": "Cortado"}
>kassa-2:{"orderId": 808, "customer": "Jasper", "coffee": "Mocha"}
>kassa-3:{"orderId": 809, "customer": "Evi", "coffee": "Ristretto"}
>kassa-1:{"orderId": 810, "customer": "Stijn", "coffee": "Cold Brew"}

## Step 5: Restore the Outage (Barista 2 Returns)

1. Restart Barista 2 in Terminal 2: podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team 

produce:
kassa-1:{"orderId": 1001, "customer": "Lars", "coffee": "Cappuccino"}
>kassa-2:{"orderId": 1002, "customer": "Yara", "coffee": "Latte"}
>kassa-3:{"orderId": 1003, "customer": "Ruben", "coffee": "Espresso"}
>kassa-1:{"orderId": 1004, "customer": "Femke", "coffee": "Flat White"}
>kassa-2:{"orderId": 1005, "customer": "Sven", "coffee": "Americano"}
>kassa-3:{"orderId": 1006, "customer": "Lieke", "coffee": "Cortado"}
>kassa-1:{"orderId": 1007, "customer": "Bram", "coffee": "Mocha"}
>kassa-2:{"orderId": 1008, "customer": "Hanna", "coffee": "Cold Brew"}
>kassa-3:{"orderId": 1009, "customer": "Wout", "coffee": "Ristretto"}
>kassa-1:{"orderId": 1010, "customer": "Anouk", "coffee": "Macchiato"}

## Step 6: Verification in Kafdrop
1. Open Kafdrop (http://localhost:9000).
2. Click on the Consumers tab and select barista-team.
3. Check the Consumer Lag: you will see that the lag is 0, meaning all incoming orders were successfully cleared by the group, regardless of who dropped out!