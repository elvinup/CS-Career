## RHEL9 Enable Root Login

If you can't login as root right away on a fresh machine, ensure you enable  this setting

```
vi /etc/ssh/sshd_confg
```

Line 40: `PermitRootlogin yes`

then do a 

```
systemctl restart sshd
```

