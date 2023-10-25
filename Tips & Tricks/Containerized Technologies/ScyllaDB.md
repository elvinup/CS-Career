## Install

This will spin up 3 scyllaDB containers, creating a 3 node cluster!

```bash
# Need to run this as root, since rootless podman cant fetch the IPAddress for some reason
podman run --name scylla-node1 -d scylladb/scylla:5.2.0 

podman run --name scylla-node2 -d scylladb/scylla:5.2.0 --seeds="$(podman inspect --format='{{ .NetworkSettings.IPAddress }}' scylla-node1)"

podman run --name scylla-node3 -d scylladb/scylla:5.2.0 --seeds="$(podman inspect --format='{{ .NetworkSettings.IPAddress }}' scylla-node1)"
```

Can use this command to verify finished (UN status on all 3 nodes)

```bash
podman exec -it scylla-node3 nodetool status  
```

## CQL Shell

To start interacting directly with the cluster

```bash
podman exec -it scylla-node3 cqlsh 
```

More basic instructions [here](https://university.scylladb.com/courses/scylla-essentials-overview/lessons/high-availability/topic/consistency-level-demo-part-1/)

## Replication Factor (FL)

Generally, can set replication factor with a concept called a `Keyspace`, which is metadata for multiple tables to follow.

General rule is use 1 keyspace per application, so usually 1 per cluster.

Creating, using, and listing the keyspace:

```cql
CREATE KEYSPACE mykeyspace WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'replication_factor' : 3};
use mykeyspace; use
DESCRIBE KEYSPACE mykeyspace;
```

## Consistency Level (CL)

Can set consistency level with 

```
CONSISTENCY QUORUM 
```

We can set consistency to ALL 
```
CONSISTENCY ALL
```

And then just stop one of the containers to see a typical INSERT operation fail since not all are available.

We would get this error:
![[Pasted image 20231022155908.png]]

## Shards

To view cores and shards on a node:

Login
```bash
podman exec -it scylla-node3 bash
```

And run this script that comes with scyllaDB's installation (or container)
```cql
./usr/lib/scylla/seastar-cpu-map.sh -n scylla
```

### Rack Grouping

ScyllaDB's `Snitch` determines how data is partitioned into availability zones like with Kafka or Elasticsearch.

[Snitch options](https://university.scylladb.com/courses/scylla-essentials-overview/lessons/architecture/topic/snitch/) say `GossipingPropertyFileSnitch` is the best one since you define which node is in which rack explicitly

Rack info configurable in `/etc/scylla/cassandra-rackdc.properties`

### Multi Datacenter

In a scyllaDB config: `DC1:3,DC2:3` means we store three copies of the data in each datacenter

#### Force Sync Between DC's

Say we have nodes 1-3 in DC1, and nodes 4-5 in DC2.

In this example, we'd run this on all 3 DC2 nodes to ensure they sync with DC1's data if we made some updates recently to it.

```cql
nodetool rebuild -- DC1
```

### Materialized View

To make an abstraction on denormalized data so they auto update from one write, can make a materialized view of that data.

```cql
CREATE MATERIALIZED VIEW get_locations AS SELECT location FROM tracking.tracking_data WHERE location IS NOT NULL AND first_name IS NOT NULL AND last_name IS NOT NULL AND timestamp IS NOT NULL PRIMARY KEY((location), first_name, last_name, timestamp);_
```

Can create intensive applications that can store and sort through data faster and more efficiently with less coding because those tasks can be handled by the ScyllaDB cluster rather than in the applications themselves using the view

## Integrations

### Spark

ScyllaDB can read data -> Spark

When reading data from ScyllaDB into Spark, data is split, and each Executor reads a chunk.

It batches information to it

If ever using [[Spark]] with ScyllaDB, follow these config tips

- Allocate 1 Spark CPU core for every 2 ScyllaDB cores;
- Allocate 2GB of RAM for every Spark CPU core.

#### Lab

In this [scylla lab](https://university.scylladb.com/courses/the-mutant-monitoring-system-training-course/lessons/using-spark-with-scylla/topic/scylla-and-spark-lab/) make these changes

```
export $HOSTNAME=127.0.1.1
```

after running ./start-spark.sh

Can view spark UI at `127.0.1.1:8080

All the lab really does is convert this table in a ScyllaDB instance

```cql
 uid | lettersinname | lname   | name
-----+---------------+---------+-------
   1 |          null | sanchez |  rick
   4 |          null |   cohle |  rust
   7 |          null |     joo |   joe
 502 |             6 |     bar |   foo
 102 |          null |     bar | byran

```

to

```cql
 uid | lettersinname | lname   | name
-----+---------------+---------+-------
   1 |            11 | sanchez |  rick
   4 |             9 |   cohle |  rust
   7 |             6 |     joo |   joe
 502 |             6 |     bar |   foo
 102 |             8 |     bar | byran
```

So we used [[Spark]] to populate those null values in a scyllaDB table to the "enriched" values they should be. 

Logic is contained in this [Scala script](https://github.com/scylladb/scylla-code-samples/blob/master/spark3-scylla4-demo/src/main/scala/LetterInNameCountEnrich.scala)

Shows a proof of concept of Spark working directly with ScyllaDB using Scala.

Doesn't seem like much, but at scale that is where spark can use parallelism with a large data set to do this more efficiently than ScyllaDB could by itself. 

Note: Stick to Java 11 instead of the latest to work with scala, or the sbt command will annoy you.