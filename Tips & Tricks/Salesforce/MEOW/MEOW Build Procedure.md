## New Stack

For a new stack, there generally isn't a vmware cluster yet because we build those ESXi hosts pending the build of the MEOW server to PXE boot their OS files to them.

This means we [Stick build](https://confluence.internal.salesforce.com/display/SFMCLINUX/Stick+Builds) MEOW as a **physical machine** for new stacks.

## Existing Stacks

TBD, but generally we can just build MEOW as a **virtual machine** if VMware clusters exist in the stack already.

## Configuration

Create a new <stack.yaml> file in the attributes folder of the sfmc_meow cookbook and upload it to chef.

[Example of dfw1q02.yaml](https://github.com/sfdc-mc-mj/LINUX.sfmc_meow/blob/W-13525360/attributes/dfw1q02.yaml)

Configure the host with chef:

```bash
sudo chef-client -o recipe[sfmc_meow]
```

### Manual Steps

There are only a couple leftover steps that are TBD how to automate yet. They just involve downloading large image files for Linux and Windows, and preparing them for MEOW's servers use.

- Download CentOS iso from somewhere and extract 
	- Can be an existing meow server or anywhere else that has it

```bash
cd /var/www/html/centos/7
curl -O http://atl1q33meow01.qa.local/centos/7/CentOS-7-x86_64-DVD-2009.iso

7z x -y /var/www/html/centos/7/CentOS-7-x86_64-DVD-2009.iso -o/var/www/html/centos/7/sources/`
```

- Download boot.wim from somewhere

```bash
cd /var/www/html/windows
curl -O http://atl1q33meow01.qa.local/windows/boot.wim
```

## Verify

If you see a cat staring at you when you do a 

```
curl localhost:3001
```

Then that means the API is up!