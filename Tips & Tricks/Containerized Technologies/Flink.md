## Install

docker-compose file. Call `podman-compose up --build -d`

This sets up just a Flink instance. 

```yaml
version: '3.8'
services:
  sql-client:
    build: sql-client/.
    command: bin/sql-client.sh
    depends_on:
      - jobmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        rest.address: jobmanager
  jobmanager:
    build: sql-client/.
    ports:
      - "8081:8081"
    command: jobmanager
    volumes:
      - flink_data:/tmp/
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        state.backend: filesystem
        state.checkpoints.dir: file:///tmp/flink-checkpoints
        heartbeat.interval: 1000
        heartbeat.timeout: 5000
        rest.flamegraph.enabled: true
        web.backpressure.refresh-interval: 10000
  taskmanager:
    build: sql-client/.
    depends_on:
      - jobmanager
    command: taskmanager
    volumes:
      - flink_data:/tmp/
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        taskmanager.numberOfTaskSlots: 3
        state.backend: filesystem
        state.checkpoints.dir: file:///tmp/flink-checkpoints
        heartbeat.interval: 1000
        heartbeat.timeout: 5000
volumes:
  flink_data:

```

UI will become available on http://localhost:8081

Entering SQL client is just 

```bash
podman-compose run sql-client
```

## Streaming Pipeline

Flink is just a processing component in the middle between a source connector and a sink or database to be be connected to.

Faker can be used to generate easy, fake data to eventually land in Kafka.

Faker -> Flink -> Kafka

For this we need to use this docker-compose file since we want Flink to be able to access Kafka on `kafka:29092`

```yaml
version: '3.8'
services:
  sql-client:
    build: sql-client/.
    command: bin/sql-client.sh
    depends_on:
      - jobmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        rest.address: jobmanager
  jobmanager:
    build: sql-client/.
    ports:
      - "8081:8081"
    command: jobmanager
    volumes:
      - flink_data:/tmp/
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        state.backend: filesystem
        state.checkpoints.dir: file:///tmp/flink-checkpoints
        heartbeat.interval: 1000
        heartbeat.timeout: 5000
        rest.flamegraph.enabled: true
        web.backpressure.refresh-interval: 10000
  taskmanager:
    build: sql-client/.
    depends_on:
      - jobmanager
    command: taskmanager
    volumes:
      - flink_data:/tmp/
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        taskmanager.numberOfTaskSlots: 3
        state.backend: filesystem
        state.checkpoints.dir: file:///tmp/flink-checkpoints
        heartbeat.interval: 1000
        heartbeat.timeout: 5000

  kafka:
    image: quay.io/strimzi/kafka:latest-kafka-3.6.0-amd64
    command:
      [
        "sh",
        "-c",
        "export CLUSTER_ID=$$(bin/kafka-storage.sh random-uuid) && bin/kafka-storage.sh format -t $$CLUSTER_ID -c config/kraft/server.properties && bin/kafka-server-start.sh config/kraft/server.properties --override advertised.listeners=$${KAFKA_ADVERTISED_LISTENERS} --override listener.security.protocol.map=$${KAFKA_LISTENER_SECURITY_PROTOCOL_MAP} --override listeners=$${KAFKA_LISTENERS}",
      ]
    environment:
      LOG_DIR: "/tmp/logs"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LISTENERS: PLAINTEXT://:29092,PLAINTEXT_HOST://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
    ports:
      - "9092:9092"
      - "9101:9101"

  akhq:
    image: tchiotludo/akhq
    environment:
      AKHQ_CONFIGURATION: |
        akhq:
          connections:
            docker-kafka-server:
              properties:
                bootstrap.servers: "kafka:29092"
    ports:
      - 28080:8080

volumes:
  flink_data:

```
### Destination Kafka Table
- Go to http://localhost:28080 to create the topic `pageviews`
- From Flink SQL client, run
```sql
CREATE TABLE pageviews_kafka (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
) WITH (
  'connector' = 'kafka',
  'topic' = 'pageviews',
  'properties.group.id' = 'demoGroup',
  'scan.startup.mode' = 'earliest-offset',
  'properties.bootstrap.servers' = 'kafka:29092',
  'value.format' = 'json',
  'sink.partitioner' = 'fixed'
);
```

### Source Faker Table

```sql
CREATE TABLE `pageviews` (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
)
WITH (
  'connector' = 'faker',
  'rows-per-second' = '10',
  'fields.url.expression' = '/#{GreekPhilosopher.name}.html',
  'fields.user_id.expression' = '#{numerify ''user_##''}',
  'fields.browser.expression' = '#{Options.option ''chrome'', ''firefox'', ''safari'')}',
  'fields.ts.expression' =  '#{date.past ''5'',''1'',''SECONDS''}'
);
```


### INSERT INTO to create a long-lived Flink job

Type this into FlinkSQL
```sql
INSERT INTO pageviews_kafka SELECT * FROM pageviews;
```


Finally, you'll see data flowing to that pageviews topic constantly directly from Flink in http://localhost:28080!

