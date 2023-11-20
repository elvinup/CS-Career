
How to setup a one node Elasticsearch and Kibana
## Install with Kibana
	
docker-compose.yml of 

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    ports:
      - 9200:9200
    restart: always
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
  kibana:
    depends_on:
      - elasticsearch
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
     - 5601:5601
    restart: always
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```

```bash
podman-compose up -d
```

This will setup Elasticsearch and Kibana version 8.11

## Streaming Pipeline

A real good use case is to setup a streaming pipeline with 
- Kafka
- Logstash
- A Python application playing the role of producer to Kafka

We can write to a topic in Kafka called "registered_user", and have that eventually written to an index in elasticsearch with the same name.

All components will be neatly containerized
### Kafka

Install [[Kafka]] with podman-compose as well
### Logstash

Now the component that connects Kafka -> Elasticsearch


Setting up the logstash.conf first
```bash
mkdir pipeline
vim pipeline/logstash.conf
```

logstash.conf
```json
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["registered_user"]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "registered_user"
  }
  stdout { codec => rubydebug }
}
```

Then run the container

```bash
podman run --rm -it -v ./pipeline/:/usr/share/logstash/pipeline/ -e XPACK_MONITORING_ENABLED=false --network host docker.elastic.co/logstash/logstash:8.11.0
```

### Python

An application that will use fake data to populate the registered_user topic

```python
from faker import Faker
from kafka import KafkaProducer
import json
import time

fake = Faker()

def get_registered_user():
    return {
        "name": fake.name(),
        "address": fake.address(),
        "created_at": fake.year()
    }

def json_serializer(data):
    return json.dumps(data).encode("utf-8")

producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         value_serializer=json_serializer)

if __name__ == "__main__":
    while 1 == 1:
        registered_user = get_registered_user()
        print(registered_user)
        producer.send("registered_user", registered_user)
        time.sleep(4)

```

Creates a new fake user every 4 seconds to Kafka, which eventually propagates to Kibana

### Kibana

From Kibana, go to Stack management -> Index Management 

Then just follow the prompt to create an index pattern on "registered_user*"

After that, we can play with the search bar UI in the big blue **Discover Index** button on top right.

## End Result

![[Pasted image 20231112175023.png]]

![[Screenshot from 2023-11-12 17-51-03.png]]

Now that's any easy way to make use of Elasticsearch for your Search Engines!