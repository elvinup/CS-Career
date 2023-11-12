
## Install

## Apache Kafka

- Kafka version 3.6
- akhq for a nice GUI to manage topics

CMAK (Cluster Manager for Kafka) still depends on Zookeeper, so instead we can use the open source tool **akhq** which may be even better than CMAK and works with Kraft.

```yaml
version: '3.6'
services:
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

```


```bash
podman-compose up -d
```

### Confluent Kafka

This installs all the Confluent Kafka suite of stuff

- 1 broker
- schema-registry
- connect
- rest-proxy
- ksqldb-server
- ksql-datagen
- ksqldb-cli
- control-center

Kinda bloated and this requires an enterprise license to scale, so screw this