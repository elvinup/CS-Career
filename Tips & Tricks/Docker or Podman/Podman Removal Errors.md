
## Remove container and images

For when your containers/images want to stick around despite trying to restart them

```
podman container rm -fa
podman system prune -fa && podman rmi -a
```

