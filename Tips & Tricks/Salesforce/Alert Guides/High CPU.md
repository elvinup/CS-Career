## Overview

High CPU alerts can be caused my multiple reasons. This could be because a stray process is hogging CPU, or too many processes are running that cumulatively eat up CPU.
  

## Validate Issue Exists

A great way to check CPU distribution is using the `top` command

Each row is a different process, and CPU percentage is indicated in the 3rd column

## Troubleshoot Issue

### Logrotate

If the `top` command show a lot of logrotate commands, that likely means there is a bug in one of the logrotate configs.

Running this will tell you which config it hangs on and has a syntax issue
```
logrotate -d /etc/logrotate.conf
```

Write an investigation ticket like [this one](https://gus.lightning.force.com/lightning/r/ADM_Work__c/a07EE00001eMtkzYAC/view) to the responsible team to resolve this

## Example Pager Duty Alert

An [InstanceDown](https://confluence.internal.salesforce.com/display/SFMCLINUX/InstanceDown+Alert+Playbook) Alert actually was caused by high cpu usage because it took resources from the node_exporter service running on the box. 

In this case, CMS team's change caused a syntax bug in `/etc/logrotate.d/pm2config` which then caused logrotate to hang and keep spawning more processes, hogging CPU.

## References


## Glossary