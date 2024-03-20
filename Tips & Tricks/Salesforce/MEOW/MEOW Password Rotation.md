The process to rotate meow server's password is fairly simple.

HAProxy is the reverse proxy in front of the MEOW API , so the password needs to be changed on haproxy, specifically `/etc/haproxy/haproxy.cfg`

This password and bcrypt has will be sourced from thycotic, and then applied with Chef using the below steps. 
## Creating Passwords

This [repo](https://github.com/jriddle-sf/generate-xkcd-password) is great for creating a simple password with a container.

- clone the repo
- cd into the folder and build the image
```
docker build -t --tag xkcd_password .
```
- run the container off the image
```
docker run --rm xkcd_password
```

It will spit out a password and, very conveniently, its bcrypt hash like this:

```json
{"password": "Icon-Frill-Hypnosis-Operation-Casualty-Gauze", "hash": "$6$lAC7rfAEWfvf4wnP$kk87l7fYViju2a12QXkoP4I8FIBqwroPXrT3c55QU/YyfYMIj5tYEJ1lRx2BiqqqOUlHroAmnoFS2GpjTKyk50"}
```

## Password in Delinea

Login to Thycotic/Delinea, and replace the passwords like this

- new `password` should go into **Linux/MEOW/meow_api**
- new `hash` should go into **Linux/MEOW/haproxy_hash**

## Chef Run

With those values set, Chef will pull those new values from Delinea and apply them to the haproxy configuration.

Run this against all MEOW servers:
```bash
knife ssh -m "$(< serverlist)" "sudo cinc-client -o recipe[sfmc_meow]"
```

This will also trigger a restart of the haproxy systemd service on each server to apply the change.

## Notify Customers

Make sure to let people know the password has changed. The slack channel `#mc-infra-buildbot` will have all the folks that would care.

Done!