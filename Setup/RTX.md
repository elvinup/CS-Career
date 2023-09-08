
Prerequisite to install RTX in [[Workstation]]

Using python as an example
## Install new tool

```
rtx global python@latest
```

can also do this, but still have to set to global to access it anyways

```
rtx install python@latest
```
## List all possible Packages

```
rtx plugins ls-remote | grep <name>
```

## Remove tool

```
rtx uninstall <tool@version>
```

## List version of specific tool

Works only after installing the plugin

```
rtx ls-remote python
```