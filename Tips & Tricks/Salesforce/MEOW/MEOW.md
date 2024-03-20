Marvelously-Easy-OS-Worker

[Design Doc](https://salesforce.quip.com/No9WAapQw5FD)

The MEOW is a PXE server to help deploy physical servers for Linux, Windows, and ESXi in Marketing Cloud

Servers that need to be PXE booted will be assumed to be in a boot loop attempting to boot off the network, until an API call to the MEOW is made for that respective server.

This does the "PXE Magic" and suddenly your server boots into the OS you want

# Table of Contents 
| How do I                | Knowledge Link                                                                                                  |
| ----------------------- | --------------------------------------------------------------------------------------------------------------- |
| Configure MEOW Host     | [MEOW VM Build Procedure](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW+VM+Build+Procedure) |
| Access the API          | [API](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#api)                                    |
| Find MEOW endpoints     | [Layout](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#layout)                              |
| Basic Usage             | [Using MEOW](https://confluence.internal.salesforce.com/display/SFMCLINUX/Using+MEOW)                           |
| Basic TroubleShooting   | [Chef Cookbook](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#chef-cookbook)                |
| Deal with Podman        | [Podman](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#podman)                              |
| Restart Service         | [Service](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#service)                            |
| Monitor DHCP Traffic    | [DHCP](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#dhcp)                                  |
| Fix Windows PXE Booting | [Windows](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#windows)                            |
| Fix ESXi PXE Booting    | [ESXi](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#esxi)                                  |
| Password Rotation       | [MEOW Password Rotation](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW+Password+Rotation)   |

## Configuration

Follow the steps in [MEOW VM Build Procedure](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW+VM+Build+Procedure)

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

The litestar library that powers this API also provides api documentation in other formats if you don't prefer swagger's layout:

- `/swagger/elements` 
- `/swagger/rapidoc`   
- `/swagger/redoc` (Can't run API calls, view only)

## Layout

There can only be 1 MEOW per pod. 

MEOW is essentially hosting a DHCP server, and we cannot have multiple DHCP/MEOWs conflicting in the same IP space like multiple stacks in a pod would be.
### QA

| Geo     | Pods    | Stacks  | MEOW                   |
| ------- | ------- | ------- | ---------------------- |
| Atlanta | atl1q01 | atl1q33 | atl1q33meow01.qa.local |
|         |         | atl1q14 | atl1q33meow01.qa.local |
| Dallas  | dfw1q02 | dfw1q34 | dfw1q02meow01.qa.local |
|         |         | dfw1q13 | dfw1q02meow01.qa.local |

### PROD

| Geo | Pods    | Stacks  | MEOW                   |
| --- | ------- | ------- | ---------------------- |
| FRA |         | fra3s50 | fra3s50meow01.xt.local |
| CDG |         | cdg3s51 | cdg3s51meow01.xt.local |
| DFW |         | dfw1s10 | dfw1s10meow01.xt.local |
| LAS | las1p03 | las1s04 | las1s04meow01.xt.local |
|     |         | las1s05 | las1s04meow01.xt.local |
| ATL |         | atl1s11 | atl1s11meow01.xt.local |
|     | atl1p04 | atl1s07 | atl1s07meow01.xt.local |
|     |         | atl1s08 | atl1s07meow01.xt.local |
| IND |         | ind1s01 | ind1s01meow01.xt.local |
|     |         | ind1s06 | ind1s06meow01.xt.local |
| IAD |         | iad4s13 | iad4s13meow01.xt.local |
|     |         | iad4s12 | iad4s12meow01.xt.local |
## Troubleshooting

### Chef Cookbook

The MEOW API is deployed with the CINC cookbook `sfmc_meow`. Start troubleshooting by running this idempotent recipe.

```bash
sudo cinc-client -o recipe[sfmc_meow]
```

### Podman 

The MEOW API is hosted inside a podman container to more easily handle the dependencies of requiring python3.10 for our API library [litestar](https://github.com/litestar-org/litestar).

Use the [Podman](https://confluence.internal.salesforce.com/display/SFMCLINUX/Podman) page for common troubleshooting techniques which will apply for the meow-api container

### Service
Generally you can just:
- SSH into the meow server with the issue
- sudo up to root, then switch to the meow user which has access to the container
```bash
sudo -s
su - meow
```
- Check the status or restart the container if it's not running
```bash
systemctl --user status meow-api
podman ps
systemctl --user restart meow-api
# etc
```

### DHCP

You may be asked to check if the DHCP server is seeing traffic from a linux/windows/esxi host. Grab the MAC address from GUS, or from `meow_url/hosts/esxi` if an esxi host.

- You can monitor the traffic by 
	- Taking the mac address
	- Doing a `grep '<mac_address>' /var/log/messages` and it should show recent dhcp traffic if things are working for the specific host you want to monitor

You can also just 

```
tail -f /var/log/messages | grep dhcpd
```

right after rebooting a host to see what messages it exchanges with the DHCP server.

Specifically look for messages for that IP in this order. 
```
D - DISCOVER
O - OFFER
R - REQUEST
A - ACK
```

If no traffic is coming through, the fix will likely require escalating to neteng to help with DHCP helpers.
### Windows 

Windows team PXE boots work similarly to how the Linux team does theirs. Both use a assign a temporary IP and then get assigned their actual IP later upon OS deployment.
#### Boot files

If PXE booting windows is causing issues, ensure these files exist at `/var/ww/html/windows/`:
- BCD
- boot.sdi
- boot.wim
- wimboot
### ESXi

ESXi operates differently from Windows and Linux because their OS is in-memory, which means every reboot of these servers require a constant reliance on DHCP in the MEOW server to regain it's IP, rather than from a disk.
#### Host Entries

Asked to check if a host entry exists? `/var/www/html/hosts/esxi` contains all the host entries from which `/etc/dhcp/esxi_dhcp_reservations.conf` is recreated per `POST /pxeboot/esxi` API call. 

Each API call also restarts dhcpd using a systemd service called dhcp-restart which simply watches the `esxi_dhcp_reservations.conf` file for changes.

POST API call giving an error 500? Check permissions of the hosts files are set to `meow` for user and group. If not, run:

```
sudo chown -R meow.meow /var/www/html/hosts/esxi 
```

#### Boot files

If PXE booting still isn't working, make sure these files exist

- `/var/lib/tftpboot/esxi/snponly64.efi`
- `/var/lib/tftpboot/esxi/tramp`

## Code

MEOW API [repo](https://github.com/sfdc-mc-mj/LINUX.Containers.meow-api/) which contains the main PXE booting logic

Chef Cookbook [repo](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/) for configuring a MEOW server with the MEOW API container

## Extra Links

- [Configuring MEOW VM demo](https://drive.google.com/file/d/1jA8SYzvLG_3mWYY-nB4bMBbxsTNUWtGn/view?usp=drive_link)