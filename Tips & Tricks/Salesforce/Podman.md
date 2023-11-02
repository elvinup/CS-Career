
# General

Podman is a tool to deploy containers, similar to docker.

## Services that use Podman


| Service    | Container Name | User      |
| ---------- | -------------- | --------- |
| Kafka      | kfk            | etbigdata |
| Kafka-Rest | rest           | etbigdata |
| Zookeeper  | zk             | etbigdata |
| MEOW       | meow-api       | meow      | 


In order to properly use `podman` commands on BigData/Kafka hosts, you must login to the host as the `etbigdata` user

## Check what services are running

```
podman ps
```

## Troubleshooting

### Container Keeps restarting

If you want to see a container's logs, but it keeps restarting, run this command to follow the logs as it comes up and down

```
podman logs --tail 50 --follow --timestamps <container_name>
```

### Restart Service

Podman can run rootless, and it can run as a systemd user process. 

If you are logged in as root, make sure you switch to the user that owns the service to access it with `su -`. Don't forget the `-` or you'll get errors trying to restart the service.

```bash
root@host$ su - <user>
user@host$ 
```

Any systemctl command must be supplied with the `--user` command. 

```bash
systemctl --user restart <container_name>
```


### Nuke the Container

Sometimes it's just better to completely obliterate your container and pull a fresh one!
#### Remove container and images

For when your restarts aren't working and podman complains about an already existing  containers/images (seems to be more an issue with podman 1.x which we're stuck with until RHEL9?). 

```bash
podman container rm -fa
podman system prune -fa && podman rmi -a
```

This is totally safe. Restarting the container with systemd will re-pull the container image from artifactory and will run the container as normal.

```bash
systemctl --user restart <container_name>
```

### Trouble using podman at all

If you get an error like this running any podman command at all:

```
error: could not get runtime: open /proc/31678/ns/user: no such file or directory
```

This command should resolve it without having to even reboot:
```
rm -rf /tmp/run-$(id -u)/
```