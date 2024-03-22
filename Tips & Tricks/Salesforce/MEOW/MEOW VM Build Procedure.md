## New Stacks

If this is a brand new stack, and VCenter isn't available yet, then make sure you follow [MEOW Physical Build Procedure](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW+Physical+Build+Procedure) instead of this page.
## Existing Stacks

If VCenter exists, MEOW can be built as a VM in **VLAN 193** (Build VLAN)
## stack.yaml Attribute

Make sure a <stack.yaml> file in the [attributes folder of the sfmc_meow cookbook](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/tree/main/attributes) exists and upload it to chef. 

VIPs team can help with the esxi section, Windows team for the windows section. 

`tftp:` should be the IP of the MEOW server itself.

[Example of dfw1q02.yaml](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/blob/W-13525360/attributes/dfw1q02.yaml)
## Build VM

- Create a Terraform app.tfvars file: [Example](https://github.com/sfdc-mc-mj/MCAPS.Terraform-AppConfig-v14/blob/main/iad4s12/linux/meow/app.tfvars) and ensure it's available in artifactory.
- Run the Build pipeline: Jenkins -> Terraform -> BuildServer.Pipeline

### 17.1 Stack Note

After the VM is built, When building a VM in a 17.1 network in build VLAN 193, ensure that you migrate the VM to a `MGT` cluster machine in VCenter. Otherwise you likely won't be able to ping or SSH into the server.

## Configuration

Run the users recipe first to setup the meow user. UID will be used for default recipe.
```bash
sudo cinc-client -o recipe[sfmc_meow::users]
```

Run the default recipe
```bash
sudo cinc-client -o recipe[sfmc_meow]
```

What cookbook `sfmc_meow` does:

- Installs podman
- Configures podman to run rootless containers
- Sets up artifactory authentication config
- Spins up the meow_api container
- Sets up DHCP and TFTP servers for PXE booting
- Configures firewalld on ports 3001 and 3002
- Sets up HAproxy on top of the MEOW API

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
## Notify Customers

Need to notify

- `#mc-vips-public`
- `#winux`

The MEOW lives in VLAN 193, while all ESXi hosts and some Windows hosts live in other VLANs.

ESXi is a special case where they need to get their IP statically from the DHCP server. 

Delegate this work to these teams and notify them that they need to request NetEng to add a DHCP helper for their subnet containing their machines to allow DHCP traffic between VLANs.

DHCP Helper request - NetEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dXwuQYAS/view)