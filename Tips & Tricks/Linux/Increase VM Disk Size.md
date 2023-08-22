
```bash
#!/bin/bash
# 1. Says what the storage device is called - Format: sdX
dmesg | tail # Let's says it's sdc

# 2. Update the partition table if required
sudo partprobe /dev/sdc

#3. Create Physical Volume
pvcreate /dev/sdc

#4. Extend Existing Volume Group 'vgesd'
vgextend vgesd /dev/sdc

#5. Extend Logical Volume within volume group
lvextend -l 100%VG /dev/vgesd/lvesd

#6. Extend file system 
xfs_growfs /dev/mapper/vgesd-lvesd

#7. Verify the size has grown with this
df -hP
```
