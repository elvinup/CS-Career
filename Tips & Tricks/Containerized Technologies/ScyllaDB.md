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

## Basic Usage

List keyspaces:
```cql
desc keyspaces;

use <keyspace>;
```

List tables in keyspace:
```cql
desc tables;
```


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

#### Global Index

How it works

Imagine we have this table

```cql
 city        | name            | dish_type               | price
-------------+-----------------+-------------------------+-------
   Reykjavik |          hakarl |  cold Icelandic starter |    16
   Reykjavik |            svid | hot Icelandic main dish |    21
      Da Lat |         banh mi |    Vietnamese breakfast |     5
 Ho Chi Minh |         bun mam |         Vietnamese soup |     8
 Ho Chi Minh |        goi cuon |  Vietnamese hot starter |     6
      Cracow | beef tripe soup |             Polish soup |     6
      Warsaw |      pork jelly |     cold Polish starter |     8
      Warsaw |     sorrel soup |             Polish soup |     5
      Warsaw |   sour rye soup |             Polish soup |     7
```

Primary key is set on city and name, so we can't just search based on dish_type.

We have to create an index like:
```cql
CREATE INDEX ON menus(dish_type);
```

Which then allows us to do 
```cql
SELECT * from menus where dish_type = 'Polish soup';
```

![[Pasted image 20231031203351.png]]

This is what's happening internally. There's 2 hops from different nodes, deriving the primary key of "Polish soup" results, and then querying the original base table with that subset of primary keys to give the illusion it's directly giving you what you want.

Note: avoid creating an index on a low cardinality (low uniqueness) column, or the index won't be as helpful and adding storage for little ROI
#### Local Index

What about getting dishes for 1 specific city?

```cql
SELECT * FROM menus WHERE city = 'Warsaw' and dish_type = 'Polish soup';
```

We could achieve this with a local index like so
```cql
CREATE INDEX ON menus((city),dish_type);
```

Global index would still work, but introduce more nodes in communication which is more latency.

Using a local index is even faster since we're working with the same node, and shard aware libraries can allow just 1 round trip instead of 2.

Note: Local index is only applicable when we're using the **primary key** in our query, because then we have to read from more than 1 node since this is now involving multiple partitions. We can only specify 1 node and use a local index if we narrow it down to 1 partition 

![[Pasted image 20231031203748.png]]
#### Materialized View vs Global Index vs Local Index

A rule of thumb:
1. For no storage overhead, filtering is the only option
2. For highly selective queries, indexing may be more beneficial than filtering
3. If most of the queries specify a single partition key:
* a local index is faster than a global index, especially with token aware load balancing policy
* filtering a single partition may be faster than an index, depending on the partition size
4. If most of the queries are multi-partition, local indexes won’t help:
* a global index may be beneficial, especially if query selectivity is expected to be high
* filtering may be better if the query selectivity is low (i.e. the result consists of most of the rows)
![[Pasted image 20231031194755.png]]

Materialized view is faster than secondary indexes for reads because it doesn't need 2 hops like Secondary Indexes might. 

Also can't use secondary indexes if you have low cardinality (low number of options/unique values), in which case stick to a basic materialized view too. Just denormalize in this case.
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

## Drivers

Always use **prepared statements**, which are basically an easier way for scyllaDB and Cassandra to cache part of your query so it knows which node to fetch your data more precisely usually based on a MD5 hash of your query parameters.

Generally always recommended to make queries much more efficient

In other words instead of using this

```python
self.session.execute(f"INSERT INTO mutant_data (first_name, last_name, address, picture_location) VALUES ('{first_name}','{last_name}','{address}','{picture_location}')")
```

use this prepared statement

```python

self.insert_ps = self.session.prepare(
   query="INSERT INTO mutant_data (first_name,last_name,address,picture_location) VALUES (?,?,?,?)"
)

def add_mutant(self, first_name, last_name, address, picture_location):
   print(f"\nAdding {first_name} {last_name}...")
   self.session.execute(query=self.insert_ps, parameters=[first_name, last_name, address, picture_location])
   print("Added.\n")
```

It is also better to **page** results as [described here](https://university.scylladb.com/courses/using-scylla-drivers/lessons/rust-and-scylla-prepared-statements-paging-and-retries/), basically lets us stream results rather than loading everything from memory at once.

## Production

It's recommended [NOT to use containers to deploy scyllaDB in Prod](https://www.scylladb.com/2018/08/09/cost-containerization-scylla/)

The extra layer of abstraction can give a 3% performance penalty for the operational convenience