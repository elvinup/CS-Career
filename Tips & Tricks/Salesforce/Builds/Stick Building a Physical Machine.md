
This example will be about the MEOW server
## Check name

If name needs to be renamed:

Write 2 stories

AssetForce update - DCEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dHc3XYAS/view)

Update port description - NetEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dR2FXYA0/view)

## Log into iLO

Use `atl1q33mkadm01.qa.local` to be able to access iLO's in Q33 or DFW environments

## Grab ISO

Grab a Centos 7 ISO or RHEL9 ISO, download it from your friendly neighborhood MEOW server:

```
http://atl1q33meow01.qa.local/centos/7/CentOS-7-x86_64-DVD-2009.iso
```

Attach to the iLO through the remote console as a local ISO

## Reboot Host and Configure

Reset host from iLO

Eventually we'll get to this page

![[Pasted image 20231031130022.png]]

Just set Installation Destination to the 500GB SSD

You can set to automatic partitioning, but better if you specify it to omit centos-home, otherwise we will end up with this from an `lsblk`

- centos-home 492.1G
- centos-root 50G
- centos-swap 4G

If you ignored this advice and clicked automatic anyways, it's okay, to fix this

```shell
umount -fl /home
lvs
lvremove /dev/centos/home
lvextend -L+492.093G /dev/centos/root
xfs_growfs /dev/mapper/centos-root
```

## Set DNS

Create a CSV, maybe infoblox.cvs with this format

```csv
HEADER-Arecord,Address,FQDN,create_ptr
Arecord, 10.232.193.3, dfw1q02meow01.qa.local, TRUE
```

Transfer over sftp to easily get it to Centro

Then do a CSV import selecting this file to import to infoblox and have it mapped to that DNS

## Update hostname

```bash
hostnamectl set-hostname dfw1q02meow01.qa.local
```
## Update Networking

```bash
cd /etc/sysconfig/network-scripts
```

Copy from another existing physical machine all the `ifcfg-*` files

Specifically update the files

```
ifcfg-bond0 # Create this if it doesn't exist already
ifcfg-eno5
ifcfg-eno6
```

```bash
systemctl restart network
```

Now we should be able to ssh into this host!

## Setup DNS

```bash
vi /etc/resolv.conf
```

Copy from DNS hosts from any server in the same stack

Example from q33
```
domain qa.local
search qa.local

nameserver 10.209.28.1
nameserver 10.209.28.2

options timeout:2
```

Need this so we can resolve hostnames like the kcap servers we want to get the cinc rpm from!
## Install Cinc

Try to bootstrap the host with the knife bootstrap command to install cinc

With cinc, this requires getting the RPM from the nearest

```bash
knife bootstrap dfw1q02meow01.qa.local -N dfw1q02meow01.qa.local -U root -P linux123 -E dfw1q02 --bootstrap-install-command "curl -O dfw1q02kcap01.qa.local/pulp/repos/SFMC/QA/centos-7/custom/chef-el7/chef-el7/Packages/c/cinc-16.18.30-1.el7.x86_64.rpm && sudo yum install -y cinc-16.18.30-1.el7.x86_64.rpm" -r 'recipe[sfmc_os_base]'
```

## Configure Host

```
sudo chef-client -o recipe[sfmc_meow]
```