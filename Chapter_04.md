# Chapter 4: Installing Kubernetes
##### Kubernetes can be installed using different configs-- there are 4 major installation types
1. All-in-One Single-Node Installation- All master and worker components are installed and running on a single node.  Useful for learning, development, and testing, shouldn't be used for production
- An example of an All-in-One Single-Node would be Minikube
2. Single-Node etcd, Single-Master and Multi-Worker Installation- Single master node which also runs single-node etcd instance, multiple worker nodes are connected to master node
3. Single-Node etcd, Multi-Master and Multi-Worker Installation- multiple-master nodes configured in HA mode, but single-node etcd instance... multiple worker nodes connected to the master nodes
4. Multi-node etcd, Multi-Master and Multi-Worker Installation- etcd is configured and clustered in HA mode, master nodes are configured in HA node, connecting to multiple worker nodes
- This is the most advanced and recommended production setup

- Once we pick installation type, need to make infrastructure decisions such as...
- Should we setup Kubernetes on bare metal, public cloud, or private cloud?
- Which underlying OS should we use?  RHEL, CoreOS, CentOS, something else?
- Which networking solution to use?
- Check Kubernetes documentation for best setup when using a learning/production environment

##### Only a few localhost installation options available to deploy single or multinode Kubernetes clusters on our workstation
1. Minikube- single-node local Kubernetes cluster
- Preferred/recommended way to create an all-in-one Kubernetes setup locally
2. Docker Desktop- single-node local Kubernetes cluster for Windows and Mac and Linux
- Docker Desktop lets you run Docker hypervisor on your machine
3. CDK on LXD- multi-node local cluster with LXD containers

##### On-Premise VMs vs On-Premise Bare Metal
- VMs: Kubernetes can be installed on VMs created via Vagrant, VMware, vSphere, KVM, or another Config Management(CM) tool in conjunction with hypervisor software
-- Different tools available to automate installation such as Ansible or kubeadm
- Bare Metal: Kubernetes can be installed on bare metal, on top of different OS' like REHL, CoreOS, Ubuntu, CentOS, Fedora, etc.
-- Most tools to install Kubernetes on VMs can be used with baremetal installations as well

##### Cloud Installation-- can occur on either Hosted Solutions or Turnkey Cloud Solutions
- Hosted Solutions: any given software is totally managed by the provider-- the user pays hosting and management charges-- some vendors include:
-- Google Kubernetes Engine (GKE)
-- Azure Kubernetes Service (AKS)
-- Amazon Elastic Container Service for Kubernetes (EKS)
-- DigitalOcean Kubernetes
-- OpenShift Dedicated
-- Platform9
-- IBM Cloud Kubernetes Service
- Turnkey Cloud Solutions: installs Kubernetes with a few commands on an underlying laas platform
-- Google Compute Engine (GCE)
-- Amazon AWS (AWS EC2)
-- Microsoft Azure (AKS)
-Turnkey On-Premise Solutions: install Kubernetes on secure internal private clouds with a few commands
--GKE On-Prem by Google Cloud
--IBM Cloud Private
--OpenShift Container Platform by Red Hat

##### Kubernetes Installation Tools/Resources
- kubeadm: secure/recommended way to boostrap a single or multi node Kubernetes cluster-- has set of building blocks to setup cluster, easily extendable to add more features
-- Doesn't support provisioning hosts
- kubespray: installs Highly Available Kubernetes clusters on AWS, GCE, Azure, OpenStack, or bare metal-- kubespray is based on Ansible and is available on most distributions on Linux
- kops: creates, deletes, upgrades, and maintains production-grade, highly available Kubernetes clusters from CLI-- can provision machines
-- AWS is officially supported, GCE support in beta, and VMware vSphere in Alpha, with other platforms planned for the future
- kube-aws: creates, upgrades, and destroys Kubernetes clusters on AWS from CLI
