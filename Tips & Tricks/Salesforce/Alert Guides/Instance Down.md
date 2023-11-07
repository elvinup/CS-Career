## Overview

Prometheus can't reach this host

## Validate Issue Exists

- Check uptime of host

```
knife ssh name:<hostname> 'uptime'
```

## Troubleshoot Issue

### Host Reachable

If the host is reachable, this usually means it's the node_exporter service that is down

```
knife ssh name:<hostname> 'sudo systemctl status node_exporter'
```

If it's down, attempt to restart it
```
knife ssh name:<hostname> 'sudo systemctl restart node_exporter'
```

Still broken? Remove the package and rerun the recipe that installs it
```
knife ssh name:<hostname> 'sudo yum remove node_exporter'
knife ssh name:<hostname> 'sudo chef-client -o recipe[prometheus_exporters::node_exporter]'
```

This should fix the node_exporter on that host

### Host Not Reachable

Usually means some HW component of the host is down.

- Check if the host has an associated ongoing case at the [Linux HPE Report](https://gus.lightning.force.com/lightning/r/Report/00OEE0000006saP2AQ/view)

If there is one already, just silence this alert with the alertmanager Jenkins pipeline

```
Jenkins -> linux -> alertmanger-silences
```

Just add the hosts to the serverlist for 3 days or so and resolve the PD.

If there's no current HPE case associated:

- Check the iLO at `<hostname>ilo.qa|xt.local`  to see if there's a broken HW component.

If so, create an HPE case with 

```
Jenkins -> automation -> hpe.new.supportcase
```
## Example Pager Duty Alert

[S12 node_exporter down](https://salesforce.pagerduty.com/incidents/Q327909PW4P46X?utm_campaign=channel&utm_source=slack)

## References


## Glossary

