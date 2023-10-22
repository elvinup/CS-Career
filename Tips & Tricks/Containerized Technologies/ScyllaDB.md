## Install

This will spin up 3 scyllaDB containers, creating a 3 node cluster!

```bash
# Need to run this as root, since rootless podman cant fetch the IPAddress for some reason
podman run --name Node_X -d scylladb/scylla:5.2.0 --overprovisioned 1 --smp 1

podman run --name Node_Y -d scylladb/scylla:5.2.0 --seeds="$(podman inspect --format='{{ .NetworkSettings.IPAddress }}' Node_X)" --overprovisioned 1 --smp 1

podman run --name Node_Z -d scylladb/scylla:5.2.0 --seeds="$(podman inspect --format='{{ .NetworkSettings.IPAddress }}' Node_X)" --overprovisioned 1 --smp 1
```

Can use this command to verify finished (UN status on all 3 nodes)

```bash
podman exec -it Node_Z nodetool status  
```

## CQL Shell

To start interacting directly with the cluster

```bash
podman exec -it Node_Z cqlsh 
```

More basic instructions [here](https://university.scylladb.com/courses/scylla-essentials-overview/lessons/high-availability/topic/consistency-level-demo-part-1/)

## Replication Factor (FL)

Generally, can set replication factor with a concept called a `Keyspace`, which is metadata for multiple tables to follow.

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

