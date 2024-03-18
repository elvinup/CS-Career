## New Stack

For a new stack, there generally isn't a vmware cluster yet because we build those ESXi hosts pending the build of the MEOW server to PXE boot their OS files to them.

This means we [Stick build](https://confluence.internal.salesforce.com/display/SFMCLINUX/Stick+Builds) MEOW as a **physical machine** for new stacks.

## Existing Stacks

TBD, but generally we can just build MEOW as a **virtual machine** if VMware clusters exist in the stack already.

## Pre-Work

The MEOW lives in VLAN 193, while ESXi hosts live in VLAN 40.

ESXi is a special case where they need to get their IP statically from the DHCP server, so we need to request NIE-TAC to add a DHCP helper to allow DHCP traffic between VLANs.

DHCP Helper request - NetEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dXwuQYAS/view)
## Configuration

Create a new <stack.yaml> file in the attributes folder of the sfmc_meow cookbook and upload it to chef.

[Example of dfw1q02.yaml](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/blob/W-13525360/attributes/dfw1q02.yaml)

Configure the host with chef:

```bash
sudo chef-client -o recipe[sfmc_meow]
```

### Manual Steps

TBD, find a better way to automate sourcing these files for the MEOW configuration

#### Windows
- Download or SCP these files from an existing MEOW server on a different stack
	- BCD
	- boot.sdi
	- boot.wim
	- wimboot

```bash
cd /var/www/html/windows
curl -O http://atl1q33meow01.qa.local/windows/BCD
curl -O http://atl1q33meow01.qa.local/windows/boot.sdi
curl -O http://atl1q33meow01.qa.local/windows/boot.wim
curl -O http://atl1q33meow01.qa.local/windows/wimboot
```

#### ESXi

**Skip this if this is a brand new stack, won't be an existing auto server to inherit host files from** 

`*auto*.xt.local` servers were the old DHCP server for ESXi machines.

Go to the corresponding stack's `auto` server to extract the hosts out of `/opt/dhcp/dhcpd-reservations.conf`

Have a file created for each host in the meow server's `/var/www/html/hosts/esxi` folder.
## Verify

If you see a cat staring at you when you do a 

```
curl localhost:3001
```

Then that means the API is up!

Add new meow stack url to the [MEOW documentation table](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW/#layout)
