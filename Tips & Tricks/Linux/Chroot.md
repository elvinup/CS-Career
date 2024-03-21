
## CentOS7
Need to change the root password to a machine you have access to? Enter Emergency mode:

- Reboot server, on the GRUB menu click `e` to edit the first entry
- Scroll towards the bottom with arrow keys (vim bindings won't work here) where `vmlinuz` is mentioned and change the argument `ro` to `rw init=/sysroot/bin/sh`
- Press `CTRL-X` to save

```bash
chroot /sysroot/
```

```bash
passwd # prompts for new root pw
```

```bash
exit # exit chroot env
reboot #reboot normally and enter new root pw
```

## RHEL9
- Reboot server, on the GRUB menu click `e` to edit the first entry
- Scroll towards the bottom with arrow keys (vim bindings won't work here) where `vmlinuz` is 
	- remove last few parameters on the kernel, until to the 512MB keyword
	- append this to that line `rw init=/bin/bash`
- Press `CTRL-X` to save

```
# check if last value is /dev/sda2, or match whatever is listed there. 
# It will change per VM
mount

# mount parititon to root, allows writing
mount -o remount,rw /dev/sda2 / 

passwd root # Change root password, or delete /etc/passwd- if it exists
```

exit
reboot