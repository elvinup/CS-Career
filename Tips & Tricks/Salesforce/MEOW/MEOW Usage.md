## Pre-Req

### Networking
For any machine you want to PXE boot, for Linux, you should ensure it's native VLAN is 193. 

This is what will allow DHCP traffic to transfer the magic `snp.efi` bootfile between the MEOW server and the new machine.

If traffic can't be found from [monitoring DHCP logs](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#dhcp), it is likely it's an underlying networking issue and you'll want to mention this in `#ma-neteng-public`'s slack channel to allow them to talk to each other.

### Boot Mode

On the physical machine's iLO page, ensure in Administration -> Boot Order is set to:

- UEFI mode
- First entry is **NIC (PXE IPv4)**

Note: The infrabuildbot pipeline in Jenkins will typically set this for you, but this is good to know if you're attempting to do this manually
## Basic Usage

Need to PXE boot a physical machine? 

Let's go through a Linux example.

Go to the MEOW for the [stack you want](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#layout) and pick the [swagger page style you want](https://confluence.internal.salesforce.com/display/SFMCLINUX/MEOW#api)

**Important!** Go to the HTTPS link over port 3002 to enable the `POST /pxeboot` endpoints.
(3001 is meant for endpoints that don't need authentication, POST won't work here.)

- MEOW endpoints in QA need to be reached through [remote app](https://confluence.internal.salesforce.com/display/MCWINFRA/MC+QA+RemoteApp+-+User+Guide) or a proper QA jumpbox that can access new QA environments
- MEOW endpoints in Prod can be reached through Centro or PRA

From there:
- Go to the OS section for the OS you want to deploy on a physical machine
- Click `POST /pxeboot/<os>`
- There's an example valid JSON body that you'll want to replace values for
	- `hostname`: FQDN of your physical machine
	- `vlan`: Check adjacent entries in infoblox for the vlan ID number this machine should reside in
	- `gateway`: Used by the linux kickstart script for networking, last IP of the subnet it's in
	- `serial_number`: serial number of machine, can be found on GUS page
	- `subnet_size`: Used by kickstart script, can be checked with infoblox
	- `os`: Choices are `centos7` and `rhel9` for linux hosts. Full choices determined by presence of ipxe scripts [in this directory](https://github.com/sfdc-mc-mj/LINUX.Containers.meow-api/tree/main/app/templates/ipxe_scripts)
		- You can also put in `shell` if you want to boot to ipxe shell to see what values are present for this machine or for general debugging if things are wonky.
	- `app`: TBD when this will be meaningful, but right now is just an extra value that can be anything
	- `meow_endpoint`: Meow server you're making this request to. Used by kickstart script.
	- `pow_endpoint`: Pick the pow server that is in the same pod/stack as your meow you're sending this request to.
	- `chef_env`: Used for choosing which chef environment this host will be bootstrapped to.
	- `dns_servers`: Put in 2 DNS servers in array format for the respective stack

Hit **Execute**

You should see a prompt to login with credentials.

These can be found in QA and Production Thycotic at:

```
Linux/MEOW/meow_api
```

Note:
Ideally, infrabuildbot's **master_build_pipeline** sources all these parameters for us through the [Declarative Data Repo](https://github.com/sfdc-mc-mj/INFRA.1PDD)

If you need to do this for many servers and don't want to use infrabuildbot, you can make a simple bash script with **curl** or a python script with the **requests** module to make a bunch of these API calls at scale.

## Development

Clone [MEOW API source code](https://github.com/sfdc-mc-mj/LINUX.Containers.meow-api)

After you `cd` into the source code directory, you can run the API server locally with:
```bash
python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8080
```

Now it's ready to test!
```
curl localhost:8080
```

Recommended to use tools like Postman to test different endpoints where you have to send a JSON payload for simplicity.