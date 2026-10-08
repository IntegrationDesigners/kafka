# Exercise 3: Consumer groups

In this exercise, we will explore how kafka ensures multiple baristas can co-operate without making duplicate  orders.
We will also learn the difference between a shared group and a unique group

---

## What Will You Learn?
* How kafka devides events over multiple consumers in the same consumer group
* How partitions decide which consumer receives which events
* the difference between a shared group and an independant group

---

## Step 1: Start Barista 1
We are going to add a property defining a fixed group name (barista-team)

1. Open a terminal
2. Start a fisrt consumer in the barista team:
	podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team
	! Don't add --from-beginning!
3. Leave it open as Barista 1

---

## Step 2: Start Barista 2

1. Open a new terminal
2. Start a second consumer in the barista team:
	podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group barista-team
	! Don't add --from-beginning!
3. Leave it open as Barista 1
   
## Step 3: Start a new producer terminal
We will be sending a series of orders to investigate how the tasks are devided

1. Open a new producer terminal
2. podman exec -it kafka kafka-console-producer --bootstrap-server localhost:29092 --topic coffee-orders --property "parse.key=true" --property "key.separator=:"
3. Send 6 different orders:

kassa-1:{"orderId": 401, "customer": "Alice", "coffee": "Cappuccino"}
>kassa-2:{"orderId": 402, "customer": "Bob", "coffee": "Espresso"}
>kassa-3:{"orderId": 403, "customer": "Charlie", "coffee": "Latte"}
>kassa-1:{"orderId": 404, "customer": "Diana", "coffee": "Americano"}
>kassa-2:{"orderId": 405, "customer": "Emma", "coffee": "Flat White"}
>kassa-3:{"orderId": 406, "customer": "Frank", "coffee": "Mocha"}
>kassa-1:{"orderId": 407, "customer": "Grace", "coffee": "Macchiato"}
>kassa-2:{"orderId": 408, "customer": "Hannah", "coffee": "Ristretto"}
>kassa-3:{"orderId": 409, "customer": "Ian", "coffee": "Cortado"}
>kassa-1:{"orderId": 410, "customer": "Julia", "coffee": "Iced Coffee"}
>kassa-2:{"orderId": 411, "customer": "Kevin", "coffee": "Cappuccino"}
>kassa-3:{"orderId": 412, "customer": "Laura", "coffee": "Espresso"}
>kassa-1:{"orderId": 413, "customer": "Mark", "coffee": "Latte"}
>kassa-2:{"orderId": 414, "customer": "Nina", "coffee": "Americano"}
>kassa-3:{"orderId": 415, "customer": "Oscar", "coffee": "Flat White"}
>kassa-1:{"orderId": 416, "customer": "Paula", "coffee": "Mocha"}
>kassa-2:{"orderId": 417, "customer": "Quinten", "coffee": "Macchiato"}
>kassa-3:{"orderId": 418, "customer": "Roos", "coffee": "Ristretto"}
>kassa-1:{"orderId": 419, "customer": "Sven", "coffee": "Cortado"}
>kassa-2:{"orderId": 420, "customer": "Tine", "coffee": "Iced Coffee"}
>kassa-3:{"orderId": 421, "customer": "Uwe", "coffee": "Cappuccino"}
>kassa-1:{"orderId": 422, "customer": "Vera", "coffee": "Espresso"}
>kassa-2:{"orderId": 423, "customer": "Wout", "coffee": "Latte"}
>kassa-3:{"orderId": 424, "customer": "Xander", "coffee": "Americano"}
>kassa-1:{"orderId": 425, "customer": "Yara", "coffee": "Flat White"}
>kassa-2:{"orderId": 426, "customer": "Zoe", "coffee": "Mocha"}
>kassa-3:{"orderId": 427, "customer": "Arne", "coffee": "Macchiato"}
>kassa-1:{"orderId": 428, "customer": "Bram", "coffee": "Ristretto"}
>kassa-2:{"orderId": 429, "customer": "Ciska", "coffee": "Cortado"}
>kassa-3:{"orderId": 430, "customer": "Dries", "coffee": "Iced Coffee"}


	
## Step 4: observe the results

1. You wil notice how the orders are devided between the twoo baristas
2. because the topic has 3 partitions, the events are devided between the active consumers in the barista-team group

2. Open your topic: coffee-orders
3. click on View Messages
4. find the correct partition where the messages are stored 

## Step 5: Check in Kafdrop
1. Go to: http://localhost:9000/
2. Go to the topic and look for the consumer ID
3. investigate the messages in the different partitions

## Step 6: Add an extyra consumer in a seperate consumer group -> analytics

1. Open a new terminal
2. Start a second consumer in the analytics team:
	podman exec -it kafka kafka-console-consumer --bootstrap-server localhost:29092 --topic coffee-orders --group analytics-team --from-beginning
	! Add --from-beginning!
3. Leave it open as analytics 1

Tip: add property --from-beginning so the analytics team can analyse from the beginning

## Extra info on the producing properties:
	- --property "parse.key=true": ensures the producer devides the event in two parts (key and value)
		-> this is requried when working with keys, else it would be seen as payload only
	- --property "key.separator=:" :  difines whic sign serves as a seperator
	
	Read more about keys: https://www.confluent.io/learn/kafka-message-key/#order-processing