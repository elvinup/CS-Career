
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
vi /etc/cinc/t
```