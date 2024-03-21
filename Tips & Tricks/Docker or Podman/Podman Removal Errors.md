
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
Will resolve the issue without reboot


## Podman Compose Networking Problems

Looks like ubuntu can't install  these required packages. Looks like we'll have to wait until it's ready, or upgrade ubuntu at some point

```shell
sudo dnf install podman podman-docker podman-plugins dnsmasq
```

## Podman container and Systemd Not working

`podman ps` only working when the --user container systemd service is off?

It's in a weird state because of the OS level systemd meow@1277 above is messed up and needs to be restarted. TODO how to keep this working persistently. Seems to get triggered by reboots.

```bash
# switch to meow user
# turn off --user meow-api service
su - meow
systemctl --user stop meow-api
# go back to root
# restart meow@1277, this seems to fix this weird state 
exit
systemctl restart meow@1277
```