
## Emergency Mode

This usually means that a disk was reassigned. 

Boot into single user mode to edit /etc/fstab to comment out drives that could be causing this to happen

Follow this [guide](https://www.tecmint.com/boot-into-single-user-mode-in-centos-7/)

- Go to GRUB menu and click `e` to edit the first entry
- Change the argument `ro` to `rw init=/sysroot/bin/sh`
- Press `CTRL-X` to save

```
chroot /sysroot/
```

```
mount -o rw,remount /
```

Now you can edit /etc/fstab without "read-only" errors.

