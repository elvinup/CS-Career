If you need to bootstrap a node manually
## New Node
To boot strap a new node follow these commands

Install Cinc
```
dnf install http://{{ pow_endpoint }}/repos/rhel9/extras/cinc-18.2.7-1.el9.x86_64.rpm -y
```

```bash
mkdir /etc/cinc
mkdir /etc/cinc/trusted_certs
```

Set up key and cert

validation.pem is needed to talk to the chef server the first time, will download new client.pem after
- Node should not already exist otherwise do
	- `knife node delete <hostname> -y`
	- `knife client delete <hostname> -y`
Take validation.pem from an existing node part of chef and place it in
```
vi /etc/cinc/validation.pem
```

Same with the trusted_cert

```
vi /etc/cinc/trusted_certs/chef_[qa|xt]_local.crt
```

```
cinc-client -E {{ chef_env }} -o recipe[rhel9_sfmc_repos]
cinc-client -E {{ chef_env }} -o recipe[databag]
cinc-client -r recipe[rhel9_sfmc_os_base]
```

## Existing Node

For bootstrapping a node that used to be in the inventory.

Delete  `/etc/cinc/client.pem`, it will look at that instead of `validation.pem` if it exists