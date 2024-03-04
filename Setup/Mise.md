Sick way to manage most languages/tools on a machine. Way faster and sleeker than OS package managers.

Prerequisite to install Mise in [[Workstation]]

```bash
curl https://mise.run | sh
~/.local/bin/mise --version
mise 2024.x.x
```

Using python as an example
## Install new tool

```
mise global python@latest
```

can also do this, but still have to set to global to access it anyways

```
mise install python@latest
```
## List all possible Packages

```
mise plugins ls-remote | grep <name>
```
## My Plugins

.mise_plugin_list
```
awscli
golang
kubectl
lua
nodejs
python
ruby
rust
vault
redis-cli
redis
maven
java
terraform

```
## Remove tool

```
mise uninstall <tool@version>
```

## List version of specific tool

Works only after installing the plugin

```
mise ls-remote python
```