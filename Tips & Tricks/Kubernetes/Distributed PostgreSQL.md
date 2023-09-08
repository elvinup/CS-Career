After installing Podman Desktop in [[Ubuntu K8s Workstation]]:

Follow [this guide](https://github.com/cloudnative-pg/cloudnative-pg/blob/2a104f9f46004413c4af58d2cf43d5233125eb6a/docs/src/quickstart.md) to install CNPG's PostgreSQL operator which makes this task way easier and more capable.

```bash
# Not necessary but just to get used to separate this from everything else in k8s
kubectl create namespace postgres

# Install Cloudnative-PG
kubectl apply -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.20/releases/cnpg-1.20.2.yaml

# Install cnpg plugin to use cli and see which is primary (only may need to do once)
curl -sSfL \ https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \ sudo sh -s -- -b /usr/local/bin

# After creating a cluster.yaml file from the above guide
kubectl -n postgres apply -f cluster.yaml

# To see which is primary (usually example-1)
kubectl cnpg status postgres-cluster-example -n postgres

# REPEAT JUST THESE 3 STEPS
# Grab postgres superuser password into clipboard
kubectl -n postgres get secret postgres-cluster-example-superuser -o jsonpath='{.data.password}' | base64 --decode | pbcopy

# Port forward to access the primary locally. From here we can use anything to connect to port 5432!
kubectl -n postgres port-forward postgres-cluster-example-1 5432:5432

# Access postgresql via CLI!
psql -h localhost -p 5432 -U postgres
```

