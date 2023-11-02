
[Design Doc](https://salesforce.quip.com/No9WAapQw5FD)

The MEOW is a PXE server to help deploy physical servers for Linux, Windows, and ESXi in Marketing Cloud

Servers that need to be PXE booted will be assumed to be in a boot loop attempting to boot off the network, until an API call to the MEOW is made for that respective server.

This does the "PXE Magic" and suddenly your server boots into the OS you want

## API

The easiest way to get familiar with all the endpoints or even to troubleshoot is to go to this URL and play with the swagger docs and try the examples in QA:
```
http://<pod>meow01.qa.local:3001/swagger
```
Note: POST /pxeboot will be restricted from running here since no auth is required here.

If you want to run POST /pxeboot, go to the HTTPS link:
```
https://<pod>meow01.qa.local:3002/swagger
```

The litestar library that powers this API also provides api documentation in two other formats if you don't prefer swagger's layout:

- `/swagger/elements` 
- `/swagger/redoc` (Can't run API calls, view only)

## Layout

There can only be 1 MEOW per pod. 

MEOW is essentially hosting a DHCP server, and we cannot have multiple DHCP/MEOWs conflicting in the same IP space like multiple stacks in a pod would be.
### QA

| Geo     | Pods    | Stacks  | MEOW                   |
| ------- | ------- | ------- | ---------------------- |
| Atlanta | atl1q01 | atl1q33 | atl1q33meow01.qa.local |
|         |         | atl1q14 |                        |
| Dallas  | dfw1q02 | dfw1q34 | dfw1q02meow01.qa.local |
|         |         | dfw1q13 |                        |

### PROD

TBD

## Troubleshooting

The MEOW API is hosted inside a podman container to more easily handle the dependencies of requiring python3.10 for our API library.

Use the [[Podman]] page for common troubleshooting techniques

## Code

MEOW API [repo](https://github.com/sfdc-mc-mj/LINUX.Containers.meow-api/)

Chef Cookbook [repo](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/)