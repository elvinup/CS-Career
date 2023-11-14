If a user want to be able to SSH into Linux server you own

```
adduser jmonk
passwd jmonk # Enter password after
```

If they want sudo privileges, add them to the wheel group

```
usermod -aG wheel jmonk
```