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

