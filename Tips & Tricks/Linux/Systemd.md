## Finding Service Systemd Unit Files

Usually they're auto installed to:

`/lib/systemd/system`

For non root controlled services:

`~/.config/systemd/user`


## Service that Restarts another Service if File Modified

First, create a `nomad-restart.service` `oneshot` service that will restart your `nomad` service. Create /etc/systemd/system/nomad-restart.service with the following content:

```
[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl restart nomad.service
```

Next, we need a `.path` unit to activate a service when a specified file is modified. Create `/etc/systemd/system/nomad-restart.path` with the following content:

```
[Path]
PathChanged=/etc/myapp/myconfig.json

[Install]
WantedBy=multi-user.target
```

Start and enable the path unit:

```
systemctl enable --now nomad-restart.path
```