A sleek way to do this is to use KIND (Kubernetes-IN-Docker) so the whole 1 node kubernetes cluster is in a docker container. 

Use podman instead of docker so it's all rootless

## Install

### Podman

```bash
sudo apt-get install podman

# Need to update the networking plugin to work with k8s
curl -O http://archive.ubuntu.com/ubuntu/pool/universe/g/golang-github-containernetworking-plugins/containernetworking-plugins_1.1.1+ds1-1_amd64.deb

sudo dpkg -i containernetworking-plugins_1.1.1+ds1-1_amd64.deb
```

### Podman Desktop

```bash
sudo apt install flatpak
sudo apt install gnome-software-plugin-flatpak

flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

flatpak install flathub io.podman_desktop.PodmanDesktop
```
### k8s tools

Install `rtx` via notes in [[Workstation]]

```bash
# TODO: Probably a sleeker way to do this with .tools-versions file
rtx global kubectl@latest
rtx global kind@latest
rtx global go@latest
rtx global minikube@latest
```

## Using k8s locally

## KIND (K8s-IN-Docker) cluster

**First time**:
Open `podman-desktop` and go to Settings -> Resources -> KIND -> Create new and choose default settings and click `Create`

**Afterwards**:
```bash
podman start kind-cluster-control-plane
```

Now you can start running common kubectl commands and it'll look at your KIND cluster as a full fledged k8s cluster!

### Stop KIND cluster

```bash
podman stop kind-cluster-control-plane
```