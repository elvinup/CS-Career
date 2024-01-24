Need to change the root password to a machine you have access to? Enter Emergency mode:

- Reboot server, on the GRUB menu click `e` to edit the first entry
- Change the argument `ro` to `rw init=/sysroot/bin/sh`
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
