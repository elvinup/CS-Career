
## Remove container and images

For when your containers/images want to stick around despite trying to restart them

```
podman container rm -fa
podman system prune -fa && podman rmi -a
```


## Trouble using podman

### error: could not get runtime: open /proc/31678/ns/user: no such file or directory

```
rm -rf /tmp/run-$(id -u)/
```
Will resolve the issue without rebooth