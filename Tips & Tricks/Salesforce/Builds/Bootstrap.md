
## Install Chef

Need to install chef from it's local kcap server

```
wget http://iad4s13kcap01.xt.local/pub/chef-16.18.0-1.el7.x86_64.rpm
```

```
yum localinstall chef-16.18.0-1.el7.x86_64.rpm -y
```


## Bootstrap command

```
knife bootstrap iad4s13mtadns03.xt.local -N iad4s13mtadns03.xt.local -E iad4s13 -r 'recipe[sfmc_os_base]'  -U root -P 'linux123' --sudo --use-sudo-password  --bootstrap-install-command -y
```
