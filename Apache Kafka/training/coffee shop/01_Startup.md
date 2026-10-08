## Environment & Connectivity

### Podman
	- Install podman or any alternative to run docker-compose
	- run:   podman compose up -d            
	- open Kafdrop UI: http://localhost:9000

### install required libs
1. Install required libraries:
podman exec -it jobmanager wget -P /opt/flink/lib/ [https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.1.0-1.18/flink-sql-connector-kafka-3.1.0-1.18.jar](https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.0.1-1.18/flink-sql-connector-kafka-3.0.1-1.18.jar)

podman exec -it taskmanager wget -P /opt/flink/lib/ [https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.1.0-1.18/flink-sql-connector-kafka-3.1.0-1.18.jar](https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-kafka/3.0.1-1.18/flink-sql-connector-kafka-3.0.1-1.18.jar)

### checkout
	Run the ex locally to avoid formatting issues
### Close
	- Don't forget to close the environment when you are done
	- podman compose down
