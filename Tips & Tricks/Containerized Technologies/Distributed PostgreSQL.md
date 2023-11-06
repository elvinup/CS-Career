After installing Podman Desktop in [[Ubuntu K8s Workstation]]:

Follow [this guide](https://github.com/cloudnative-pg/cloudnative-pg/blob/2a104f9f46004413c4af58d2cf43d5233125eb6a/docs/src/quickstart.md) to install CNPG's PostgreSQL operator which makes this task way easier and more capable.

Steps to re-setup (usually need to post system reboot)

- Open Podman Desktop
- Delete original KIND cluster, it's corrupted

If above is buggy just run 

```bash
KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster
```
- Spin up a new one

```bash
# Not necessary but just to get used to separate this from everything else in k8s
kubectl create namespace postgres

# Install Cloudnative-PG
kubectl apply -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.20/releases/cnpg-1.20.2.yaml

# Install cnpg plugin to use cli and see which is primary (only may need to do once)
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin

# After creating a cluster.yaml file from the above guide
kubectl -n postgres apply -f cluster.yaml

# To see which is primary (usually example-1)
kubectl cnpg status postgres-cluster-example -n postgres
```

## Access Locally
```bash
# In 1 window: Port forward to access the primary locally. From here we can use anything to connect to port 5432!
kubectl -n postgres port-forward postgres-cluster-example-1 5432:5432

# REPEATING JUST THESE 3 STEPS will work until you reboot your system
# Grab postgres superuser password into clipboard
kubectl -n postgres get secret postgres-cluster-example-superuser -o jsonpath='{.data.password}' | base64 --decode | pbcopy


# In 2nd window: Access postgresql via CLI! Disable SSL, less buggy
psql "sslmode=disable" -h localhost -p 5432 -U postgres
```

## Common psql Commands

- Create DB: `CREATE DATABASE test;`
- Connect to DB: `\c test`
- Create TABLE: 
```
CREATE TABLE index_demo (                                                 
    name VARCHAR(20) NOT NULL,                                                   
    age INT,                                                                     
    pan_no VARCHAR(20),                                                          
    phone_no VARCHAR(20)                                                         
);
```
- Insert into Table:
```
INSERT INTO index_demo VALUES ('elvin', 26, 'IPOET0935V', '12199854739');
```

