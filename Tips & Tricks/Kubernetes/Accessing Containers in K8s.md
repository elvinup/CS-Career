
Using the setup and example from [[Distributed PostgreSQL]]

## Local Machine

With a KIND cluster, this is just a container we can go inside

```bash
podman exec -it kind-cluster-control-plane /bin/bash
```

## Inside KIND

From within the KIND container, it is running k8s, so we have access to `kubectl`

To get all the current pods (within a specific namespace)

```bash
kubectl get pods -n postgres
```

## Finding Containers in Pods

In my case, there were 3 different pods, 1 for each postgreSQL instance. We'll select the first one it listed which happened to be `postgres-cluster-example-1`

Within pods, there are containers inside. 

To visit inside that container, we must grab the name of it which kubectl can provide with this:

```bash
kubectl -n postgres get pod postgres-cluster-example-1 -o json | jq '.spec.containers[].name'
```

In this case, the container name was `postgres`

## Visiting inside a container inside a k8s pod

Basically after the `-it` flag, you have to state the pod you want to go into, and the container you want with the `-c` flag

```bash
kubectl -n postgres exec -it postgres-cluster-example-1 -c postgres /bin/bash
```

Now we can explore inside this container! Makes your local k8s feel at least a bit less abstract.