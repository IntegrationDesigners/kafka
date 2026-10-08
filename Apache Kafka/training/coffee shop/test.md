## Possible solutions

### 1 topic vs multiple topics for the producer

1. Option 1:
	* topic: coffee-orders-v2
	* create a flink job which filters by city and stores the events to a dedicated topic -> barista team get their topic from flink
	* Pro: 
		- no impact for producer when new cities join
		- transparent for consumers, they get their own topic so no risk for impact
	* Cons:
		- More complex flink work for routing -> requires skills
		- per city a new sink table
		
	* Source tabel:
		CREATE TABLE coffee_orders (
			city STRING,
			kassa_id STRING,
			orderId BIGINT,
			customer STRING,
			coffee STRING,
			proctime AS PROCTIME()
		) WITH (
			'connector' = 'kafka',
			'topic' = 'coffee-orders-vlaanderen',
			'properties.bootstrap.servers' = 'kafka:29092',
			'properties.group.id' = 'flink-vlaanderen-group',
			'scan.startup.mode' = 'earliest-offset',
			'format' = 'json'
		);
	
	* Sink tabellen (routing for barista teams & analytics)
		-- Sink for baristas in Antwerpen
		CREATE TABLE antwerpen_barista_queue (
			city STRING,
			kassa_id STRING,
			orderId BIGINT,
			customer STRING,
			coffee STRING
		) WITH (
			'connector' = 'kafka',
			'topic' = 'barista-orders-antwerpen',
			'properties.bootstrap.servers' = 'kafka:29092',
			'format' = 'json'
		);

		-- Sink for baristas in Hasselt
		CREATE TABLE hasselt_barista_queue (
			city STRING,
			kassa_id STRING,
			orderId BIGINT,
			customer STRING,
			coffee STRING
		) WITH (
			'connector' = 'kafka',
			'topic' = 'barista-orders-hasselt',
			'properties.bootstrap.servers' = 'kafka:29092',
			'format' = 'json'
		);
		
		-- Sink for analytics
		CREATE TABLE city_coffee_stats_output (
			city STRING,
			coffee STRING,
			total_orders BIGINT,
			window_start TIMESTAMP(3),
			window_end TIMESTAMP(3)
		) WITH (
			'connector' = 'kafka',
			'topic' = 'city-coffee-stats',
			'properties.bootstrap.servers' = 'kafka:29092',
			'format' = 'json'
		);
	* STart flink jobs
		INSERT INTO antwerpen_barista_queue
		SELECT city, kassa_id, orderId, customer, coffee
		FROM coffee_orders
		WHERE city = 'antwerpen';
		
		INSERT INTO hasselt_barista_queue
		SELECT city, kassa_id, orderId, customer, coffee
		FROM coffee_orders
		WHERE city = 'hasselt';
		
		INSERT INTO city_coffee_stats_output
		SELECT 
			city,
			coffee,
			COUNT(orderId) AS total_orders,
			window_start,
			window_end
		FROM TABLE(
			TUMBLE(TABLE coffee_orders, DESCRIPTOR(proctime), INTERVAL '1' MINUTE)
		)
		GROUP BY window_start, window_end, city, coffee;
		
2. Option 2
	* Create a producer per city and a topic per city
	* topic: coffee-orders-antwerpen, coffee-orders-hasselt
	* Pro:
		- Isolation at source -> no risk at mixing data from different shops
		- no flink needed for routing
	* Con:
		- flink analyses become much more complex
		- new city means update of live flink jobs