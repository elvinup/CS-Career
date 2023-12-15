
## Testing in QA

Comment these lines out of the `Dockerfile`(TODO: Figure out how to get rid of this step)
```
# Copy over modified pip conf to container
#COPY ./etc/pip.conf ./etc/pip.conf
#COPY ./etc/debian.list ./etc/apt/sources.list
```

Build the container locally
```bash
docker build -t meowapi .
```

Run the container in  `~/src/containers/LINUX.Containers.meow-api/app`

```bash
docker run --rm -d -p 127.0.0.1:8080:80 --name meow-api -e MEOW=$(uname -n) -v $PWD/templates/esxi_hosts:/pxe/esxi -v $PWD/templates:/pxe/dhcp -v $PWD/templates:/var/log/meow meowapi
```

This will create files in the templates folder, remove after finishing
