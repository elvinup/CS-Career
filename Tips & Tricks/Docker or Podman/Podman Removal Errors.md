
## Remove container and images
```
podman container rm -fa
podman system prune -fa && podman rmi -a
```