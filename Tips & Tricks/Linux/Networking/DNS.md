Try to ping the gateway

For example, for server 10.209.48.146/22, ping 10.209.51.254 which is the gateway.

```
ping 10.209.51.254
```

Make sure this is the correct gateway by doing a 

```
ip route
```

This will show if it's going through the gateway properly or not. 

PRO TIP: Check an adjacent server in the same subnet and do an `ip route` to check its configuration.

## Using Netcat

nc -v 10.209.51.254 port 53