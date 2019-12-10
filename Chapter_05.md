# Chapter 5: Minikube-- Local Single-Node Kubernetes Cluster
- Type-2 Hypervisor should be installed on local workstation to run in conjunction with Minikube to get the most out of Minikube
-- Doesn't mean we have to create any VMs with guest operating systems with this Hypervisor
- Minikube builds all of its infrastructure as long as the Type-2 Hypervisor is installed
-- Minikube invokes Hypervisor to create a single VM which hosts a single-node Kubernetes cluster, so we need to ensure we have the needed specs to build the environment
kubectl- a binary used to access/manage any Kubernetes cluster
- Installed after the minikube installation-- causing warning during Minikube initialization we can disregard
- Type-2 Hypervisor- VirtualBox for KVM for Linux, VirtualBox, HyperKit, or VMware Fusion for Mac, and VirtualBox or Hyper-V for 
- Minikube does support vm-driver=none potion that runs Kubernetes components directly on the host OS and not inside a VM
-- With this option a Docker install is required and a Linux OS on the local workstation, but no hypervisor install
-- If you do this, be sure to specify a bridge network for Docker, otherwise it might change between network restarts, causing loss of connectivity to your cluster
- VT-x/AMD-v virtualization must be enabled on the local workstation in BIOS
- Internet connection on first Minikube run needed to download packages, dependencies, updates, and pull images needed to initialize the cluster
-- Subsequent runs will need a connection when new images need to be pulled from a container repo or when deployed containerized apps need it-- once this is done you can reuse it without an internet connection

- Minikub CRI-O can be started by running `minikube start --container-runtime=cri-o`
-- Start Minikube with CRI-O as container runtime instead of Docker 
- Log into Minikube's VM by running `minikube ssh`
- List container by running `sudo docker container ls`
- List container created via CRI-O with `sudo runc list`

#### Installing on Linux
1. Install VirtualBox Hypervisor-- add the source repo for the bionic distro (Ubuntu 18.04), download, and register the public key, update, and install
```bash
$ sudo bash -c 'echo "deb https://download.virtualbox.org/virtualbox/debian bionic contrib" >> /etc/apt/sources.list'
$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install -y virtualbox-6.0
```
2. Install Minikube-- can replace `/v1.0.1/` with `/latest/` to download latest version instead of specified version
```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.0.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
3. Start Minikube by typing the `minikube start` comman-- disregard unable to read docker/config errors and no matching credentials warnings
4. Check status by running `minikube status`
5. Stop minikube by running `minikube stop`


#### Installing on Mac
1. Install VirtualBox by downloading and installing the .dmg package
2. Install Minikube
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.0.1/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
3. Can then start, check status, and stop minikube with same commands as above


#### Installing on Window
1. Install VirtualBox by downloading and installing the .exe package, specifically minikube-installer.exe, not minikube-windows-amd64.exe
2. Install Minikube by downloading latest release from Minikube release page, once downloaded make sure it's added to `PATH`
3. Can then start, check status, and stop minikube with same commands as above
