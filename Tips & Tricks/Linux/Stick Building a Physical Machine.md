## Check name

If name needs to be renamed:

Write 2 stories

AssetForce update - DCEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dHc3XYAS/view)

Update port description - NetEng - : [Example](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001dR2FXYA0/view)

## Log into iLO

Use `atl1q33mkadm01.qa.local` to be able to access iLO's in Q33 or DFW environments

## Grab ISO

Grab a Centos 7 ISO or RHEL9 ISO, download it locally from the Q33 MEOW

```
http://atl1q33meow01.qa.local/centos/7/CentOS-7-x86_64-DVD-2009.iso
```

Attach to the iLO through the remote console as a local ISO

## Reboot Host and Configure

Reset host from iLO

Eventually we'll get to this page

![[Pasted image 20231031130022.png]]

Just set Installation Destination to the 500GB SSD

Can set to automatic partitioning, but we will end up with this from an `lsblk`

- centos-home 492.1G
- centos-root 50G
- centos-swap 4G

To fix this

```shell
umount -fl /home
lvs
lvremove /dev/centos/home
lvextend -L+492.093G /dev/centos/root
xfs_growfs /dev/mapper/centos-root
```

## Set DNS

Create a CSV like:

## Update hostname

## Install Cinc

## Configure Host