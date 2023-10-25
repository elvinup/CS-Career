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