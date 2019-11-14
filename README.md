# Kubernetes

## Chapter 1: Container Orchestration
Define concept of container orchestration
-Container orchestration groups multi-container applications together at scale

Explain reasons for container orchestration:
-Better performance and tools, such as binding containers and storage, keeping resource usage in-check and optimizing as needed, and scheduling containers to run ond ifferent hosts

Different container orchestration options
-Amazon Elastic Container Service hosted by AWS to run Docker containers at scale
-Azure Container Instances hosted by Microsoft Azure is basic container orchestration service
-Azure Service Fabric is open source container orchestrator provided by Msft Azure
-Kubernetes is open source tool started by Google and is part of Cloud Native Computing Foundation
-Marathon is framework to run containers at scale on Apache Mesos
-Nomad is container orchestrator from HashiCorp
-Docker Swarm is container provider by Docker Inc, part of Docker Engine

Different container orchestration deployment options
-VMs, on-premise, baremetal, workstation, laptop, datacenter, AWS, Google Cloud, Azure Cloud


-Containers: Application-centric way to deliver high-performing, scalable apps on your choice of infrastructure
-Container Images: wrap application code, runtime, and dependencies ina  pre-defined format
-Container Runtimes: use the pre-packaged images to create one or more containers
--Examples of Container Runtimes are runC, containerd, and rkt, are all good at running containers on a single host
-Container Orchestrator:  A single controller/management unit/tool that groups hosts together to form a cluster and help us meet the below requirements needed for a PROD environment:
--Fault-tolerant, scalable, uses resources optimally, can automatically discover and communicate with other apps, accessible from the external world, can update/rollback without any downtime
--Different container orchestrators include Docker Swarm, Kubernetes, Mesos Marathon, Amazon ECS, Hashicorp Nomad
---Differences explored here:  https://www.edx.org/course/introduction-to-cloud-infrastructure-technologies
--Container Orchestrators make things easier by able to:
1. Bring multiple hosts together and make them part of a cluster
2. Schedule containers to run on different hosts
3. Help containers running on one host reach out to containers running on other hosts in the cluster
4. Bind containers and storage
5. Bind containers of similar type, such as services, to a higher-level construct so there are less individual containers to manage
6. Keep resource usage in-check adn optimize as needed
7. Allows secure access to apps running inside containers

-Container Orchestrators can be deployed on bare metal, VMs, on-premise, or on a cloud of our choice.
--For example, it can be deployed on workstation/laptop, in a datacenter, on AWS, on OpenStack
--One-click installers exist to set up Kubernetes on the cloud, such as Google Kubernetes Engine on Google Cloud, or Azure Container Service on MSFT Azure

-Container image bundles application, runtime, and dependencies, which is then used to create an isolated executable environment, aka a container
-Containers can be deployed froma  given image on the platform of our choice, such as desktops, clouds, VMs, etc.


## Chapter 2: Kubernetes
Define Kubernetes
-Kubernetes: open-source system for automating/managing deployment, scaling, and management of containerized applications
-Kubernetes Case Studies: https://kubernetes.io/case-studies/

Benefits of Kubernetes
1. Features mentioned below
2. Portable and extensible
3. Can be deployed on environment of our choice such as VM or bare metal or local
4. Modular and pluggable architecture
5. Can write custom APIs/plugins to extend functionalities
6. Thriving community with many commits to code and meetups


Kubernetes Features
1. Automatical Binpacking: automatically schedules containers based on resource usage and constraints without disabling the containers
2. Self-healing: automatically replaces/reschedules the containers from failed nodes-- also kills/restarts containers that don't respond to health checks based on rules put in place
3. Horizontal Scaling: can automatically scale apps based on resource usage like CPU/RAM, can also support dynamic scaling based on customer metrics
4. Service Discovery and Load Balancing: groups sets of containers and refers to them via a DNS (Domain Name System)-- DNS is also called a service, and services can be discovered automatically
5. Automated rollouts/rollbacks without downtime
6. Secrets/configuration management: can manage secrets/configuration details for app without rebuilding the respective images, secrets let us share confidential data to app without exposing it to the stack configuration like on GitHub with .gitignore
7. Storage Orchestration: can automaticalyl mount local, external, and storage solutions to containers in a seamless manner based on software-defined storage (SDS)
8. Batch Execution


Evolution of Kubernetes
--Kubernetes is also referred to as k8s, 8 characters between k and s
--written in Go, inspired by Google Borg, donated by Google to CNCF
--Has new releases every 3 months generally 
-Kubernetes features/objects that can be traced to Borg are API servers, Pods, IP-per-Pod, Services, Labels


Explain what Cloud Native Computing Foundation (aka CNCF) does
-Hosted by the Linux Foundation, aims to accelerate the adoption of containers, microservices, and cloud-native applications
-Hosts a set of projects and provides resources to each project, even though each project operates independently
-For Kubernetes, provides neutral home for trademark and enforces proper use, provides license scanning, offers legal guidance on patent/copyright issues, creates open source training, curriculum, and certification, supports ad hoc activities, hosts activities and more


## Chapter 3: Kubernetes Architecture
Discuss Kubernetes Architecture
-At high level, Kubernetes has one or more master nodes, one or more worker nodes, and distributed key-value store, like etcd
-Master Node: responsible for managing Kubernetes cluster and is the entry point for all admin tasks
--Can communicate with MN via CLI, GUI, or APIs
--There can be multiple Master Nodes for fault tolerance purposes-- other MNs would be in a High Availability (HA) mode, with only one being the leader and performing all the operations
-Worker Node: Machine, VM or physical, which runs applications using Pods and is controlled by the MN
-Pod: Scheduling unit that is collection of one or more containers which are always scheduled together
-Pods are scheduled on the worker nodes, which have the tools to run and connect them
-Access applications from external world via WNs and not MNs



Explain different components for master and worker nodes
-MN has following components:
1. API Server: admin tasks performed via API server in MN, operator sends REST commands to API which processes/validates reqs
--After executing reqs, resulting state of the cluster is stored in the ETCD
2. Scheduler: schedules work to different worker nodes using resource usage info for each worker node
--Schedules work in terms of Pods and Services
--Knows about contraints in place, such as scheduling work on a node that has the label `disk===ssd` set
--Before scheduling work, scheduler taks into account quality of service requirements, data locality, affinity, anti-afinity, etc
3. Controller Manager: manages different non-terminating control loops, regulating the state of the cluster
--Each control loop knows about the desired state of the objects managed, and watches their current state through API server
-If current state of objects managed by control loop doesn't match desired state, control loop corrects to make sure current state === desired state
-kube-controller-manager runs controllers responsible to act when nodes become unavailable, to ensure pod counts are as expected, to create endpoints, service accounts, and API access tokens
-cloud-controller-manager runs controllers responsible for interacting with underlying infrastructure of a cloud provider when nodex are unavailable, to manage storage volumes when provided by cloud service, and to manage load balancing and routing
4. ETCD: Distributed key-value store that is used by Kubernetes to manage cluster state, and is connected to all Master Nodes
--Can be configured externally, in which case MNs would connect to it

-WN has following components:
1. Container Runtime: runs and manages container's lifecycle, some container runtimes are containerd, rkt, and lxd
2. Kubelet: Agent that runs on each WN and communicates with MN, receives Pod definition primarily through API, and runs containers associated with Pod
--Also makes sure that containers that are part of Pods are healthy at all times
--Connects to container runtime using Container runtime Interface (CRI)
---CRI consistes of protocol buffers, gRPC API, and libraries
--Kubelet connects to CRI shim to perform container/image operations
--CRI implements ImageService and RuntimeService
---ImageService repsonsible for image-related operations
---RuntimeService responsible for Pod and container-related operations

3. Kubelet CRI Shims: dockershim- containers are created using Docker installed on WNs, internally, Docker uses containerd to create and manage containers
--cri-containerd: lets use use Docker's smaller offspring `containerd` to create/manage containers
--CRI-O: enables any Open Container Initiative (OCI) compatible runtimes with Kubernetes

4. Kube-proxy: network proxy running on each WN and listens to API server for each Service endpoint creation/deletion
--For each Servie endpoint, kube-proxy sets up the routes so that it can reach to it
--Service groups related Pods, and when accessed, load balances to them
---Connect to Service isntead of to Pods directly


Discuss cluster state management with etcd
-ETCD
-etcd is based on Raft Consensus Algorithm, which allows a collection of machines to work as a coherent group that can survive failures of some members

Networking Challenges-- microserves based apps rely on networking to mimic tight-coupling once available
1. Container-to-container communication inside Pods
2. Pod-to-Pod communication on the same node and across cluster nodes
3. Pod-to-Service communication within the same namespace and across cluster namespaces
4. External-to-Service communication for clients to access applications in a cluster

Review Kubernetes Network setup requirements
1. Unique IP assigned to each Pod-- can be done with either Container Network Model (CNM, propposed by Docker) or Container Network Interface (CNI proposed by CoreOS).... Kubernetes uses CNI to assign IPs
--Container runtime offloads IP assignment to CNI, which connects to configured plugin such as Bridge or MACvlan, to get the IPs
-- Once IP is given by plugin, CNI forwards it back to requested container runtime
2. Containers in Pod can communicate with each other
--Network Namespace: all container runtimes create an isolated network entity for each container that it starts
--Inside Pod, containers share network namespaces so they can reach to each other via localhost
3. Pod is able to communicate with other Pods in the cluster-- Pods can be scheduled on any node in a clustered environment
--We need to ensure that Pods can communicate across nodes and that all nodes should be able to reach any Pod
--Kubernetes puts condition that there shouldn't be any Network Address Translation(NAT) while Pod-toPod communication across hosts occurs
---This is done via routable pods/nodes using the underlying physical infrastructure like Google Kubernetes Engine
---OR done by using Software Defined Networking like Flannel, Weave, Calico, etc.
4. If configured, application deployed inside a Pod is accessible from the external world-- by exposing services to external world, we can access applications from outside the cluster

Communication Across Nodes
-Pods are scheduled on nodes randomly in a Kubernetes cluster
--Regardless of host node, Pods are expected to communicate with all other pods in the cluster without implementing a Network Address Translation (NAT)
---FUNDAMENTAL REQUIREMENT OF ANY NETWORK IMPLEMENTATION IN KUBERNETES
--Kubernetes network model aims to reduce complexity, treats Pods as VMs ona network where each VM receives an IP-- thus assigning each Pod an IP
--This model is called IP-per-Pod, ensures proper Pod-to-Pod communication
-Containers share Pod's network namespace and must coordinate ports assignment inside the Pod just as apps would on a VM
--Must do this while communicating with each other on localhost inside the Pod
--Containers are integrated with overall Kubernetes network model through use of Container Network Interface (CNI), supported by CNI plugins
---CNI is a set of specification/libraries which lets plugins configure networking for containers
---There are a few core plugins, most are 3rd party Software Defined Networking (SDN) solutions implementing the Kubernetes networking model
---Some CNIs also offer support for Network Policies
---Flannel, Weave, and Calico are a few SDN solutions available for Kubernetes clusters
--Container runtime offloads IP assignment to CNI which connects underlying configured plugin to get IP address
---Once IP is given, CNI forwards it back to requested container runtime

Pod-to-External World Communication
-To deploy containerized apps running in Pods in a Kubernetes cluster, it needs accessibility from the outside world
--Kubernetes enables external accessibility through services
---services: complex consutrcts which encapsulate networking rules definitions on cluster nodes
--By exposing services to the external world with kube-proxy, apps become accessible from outside the cluster over a virtual IP

## Chapter 4: Installing Kubernetes
Kubernetes can be installed using different configs-- there are 4 major installation types
1. All-in-One Single-Node Installation- All master and worker components are installed and running on a single node.  Useful for learning, development, and testing, shouldn't be used for production
-An example of an All-in-One Single-Node would be Minikube
2. Single-Node etcd, Single-Master and Multi-Worker Installation- Single master node which also runs single-node etcd instance, multiple worker nodes are connected to master node
3. Single-Node etcd, Multi-Master and Multi-Worker Installation- multiple-master nodes configured in HA mode, but single-node etcd instance... multiple worker nodes connected to the master nodes
4. Multi-node etcd, Multi-Master and Multi-Worker Installation- etcd is configured and clustered in HA mode, master nodes are configured in HA node, connecting to multiple worker nodes
-This is the most advanced and recommended production setup

Once we pick installation type, need to make infrastructure decisions such as...
-Should we setup Kubernetes on bare metal, public cloud, or private cloud?
-Which underlying OS should we use?  RHEL, CoreOS, CentOS, something else?
-Which networking solution to use?
-Check Kubernetes documentation for best setup when using a learning/production environment

Only a few localhost installation options available to deploy single or multinode Kubernetes clusters on our workstation
1. Minikube- single-node local Kubernetes cluster
-Preferred/recommended way to create an all-in-one Kubernetes setup locally
2. Docker Desktop- single-node local Kubernetes cluster for Windows and Mac and Linux
-Docker Desktop lets you run Docker hypervisor on your machine
3. CDK on LXD- multi-node local cluster with LXD containers

On-Premise VMs vs On-Premise Bare Metal
-VMs: Kubernetes can be installed on VMs created via Vagrant, VMware, vSphere, KVM, or another Config Management(CM) tool in conjunction with hypervisor software
--Different tools available to automate installation such as Ansible or kubeadm
-Bare Metal: Kubernetes can be installed on bare metal, on top of different OS' like REHL, CoreOS, Ubuntu, CentOS, Fedora, etc.
--Most tools to install Kubernetes on VMs can be used with baremetal installations as well

Cloud Installation-- can occur on either Hosted Solutions or Turnkey Cloud Solutions
-Hosted Solutions: any given software is totally managed by the provider-- the user pays hosting and management charges-- some vendors include:
-- Google Kubernetes Engine (GKE)
-- Azure Kubernetes Service (AKS)
-- Amazon Elastic Container Service for Kubernetes (EKS)
-- DigitalOcean Kubernetes
-- OpenShift Dedicated
-- Platform9
-- IBM Cloud Kubernetes Service
-Turnkey Cloud Solutions: installs Kubernetes with a few commands on an underlying laas platform
-- Google Compute Engine (GCE)
-- Amazon AWS (AWS EC2)
-- Microsoft Azure (AKS)
-Turnkey On-Premise Solutions: install Kubernetes on secure internal private clouds with a few commands
--GKE On-Prem by Google Cloud
--IBM Cloud Private
--OpenShift Container Platform by Red Hat

Kubernetes Installation Tools/Resources
-kubeadm: secure/recommended way to boostrap a single or multi node Kubernetes cluster-- has set of building blocks to setup cluster, easily extendable to add more features
--Doesn't support provisioning hosts
-kubespray: installs Highly Available Kubernetes clusters on AWS, GCE, Azure, OpenStack, or bare metal-- kubespray is based on Ansible and is available on most distributions on Linux
-kops: creates, deletes, upgrades, and maintains production-grade, highly available Kubernetes clusters from CLI-- can provision machines
--AWS is officially supported, GCE support in beta, and VMware vSphere in Alpha, with other platforms planned for the future
-kube-aws: creates, upgrades, and destroys Kubernetes clusters on AWS from CLI


## Chapter 5: Minikube-- Local Single-Node Kubernetes Cluster
Type-2 Hypervisor should be instlaled on local workstation to run in conjunction with Minikube to get the most out of Minikube
-Doesn't mean we have to create any VMs with guest operating systems with this Hypervisor
Minikube builds all of its infrastructure as long as the Type-2 Hypervisor is installed
-Minikube invokes Hypervisor to create a single VM which hosts a single-node Kubernetes cluster, so we need to ensure we have the needed specs to build the environment
kubectl- a binary used to access/manage any Kubernetes cluster
-Installed after the minikube installation-- causing warning during Minikube initialization we can disregard
Type-2 Hypervisor- VirtualBox for KVM for Linux, VirtualBox, HyperKit, or VMware Fusion for Mac, and VirtualBox or Hyper-V for 
-Minikube does support vm-driver=none potion that runs Kubernetes components directly on the host OS and not inside a VM
--With this option a Docker install is required and a Linux OS on the local workstation, but no hypervisor install
--If you do this, be sure to specify a bridge network for Docker, otherwise it might change between network restarts, causing loss of connectivity to your cluster
VT-x/AMD-v virtualization must be enabled on the local workstation in BIOS
Internet connection on first Minikube run needed to download packages, dependencies, updates, and pull images needed to initialize the cluster
-Subsequent runs will need a connection when new images need to be pulled from a container repo or when deployed containerized apps need it-- once this is done you can reuse it without an internet connection

Minikub CRI-O can be started by running `minikube start --container-runtime=cri-o`
-Start Minikube with CRI-O as container runtime instead of Docker 
-Log into Minikube's VM by running `minikube ssh`
-List container by running `sudo docker container ls`
-List container created via CRI-O with `sudo runc list`



### Installing on Linux
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


### Installing on Mac
1. Install VirtualBox by downloading and installing the .dmg package
2. Install Minikube
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.0.1/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
3. Can then start, check status, and stop minikube with same commands as above


### Installing on Window
1. Install VirtualBox by downloading and installing the .exe package, specifically minikube-installer.exe, not minikube-windows-amd64.exe
2. Install Minikube by downloading latest release from Minikube release page, once downloaded make sure it's added to `PATH`
3. Can then start, check status, and stop minikube with same commands as above



## Chapter 6: Accessing Minikube
kubectl-- CLI tool to access Minikube Kubernetes cluster
-Manages cluster resources and applications
-Can be used standalone or as part of scripts and automation tools
-Once all required credentials and cluster access points have be configured for kubectl, it can be used remotely from anywhere to access a cluster
-Receives configuration from Minikube Kubernetes cluster access-- other cluster setups may need to have the acces points and certificates configured so it meets kubectl requirements
-Recommended to keep `kubectl` at same version with the Kubernetes run by MiniKube
Kubernetes Dashboard-- web-based UI to interact with the cluster, manage resources and containerized apps
curl-- commands you can write to access cluster via APIs with the proper credentials
-Can connect to API using endpoints and send commands to it

HTTP API space of Kubernets is divided into 3 groups:
1. Core Group (/api/v1)-- group includes objects such as Pods, Services, nodes, namespaces, configmaps, secrets, etc.
2. Named Group-- group includes objects in `/apis/$NAME/$VERSION` format-- different API versions imply different levels of stability and support
-Alpha: may be dropped at any point in time without notice, IE `/apis/batch/v2alpha1`
-Beta: well-tested, but semantics of objects may change in incompatible ways in a subsequent beta/stable release, IE `/apis/certificates.k8s.io/v1beta1`
-Stable: appears in released software for many subsequent versions, IE `/apis/networking.k8s.io/v1`
3. System-wide-- group conssits of system-wide API endpoints such as `/healthz`, `/logs`, `/metrics`, `/ui`, etc.
-Can either connect to API directly calling teh API endpoints or through the CLI/Web UI


### Installing kubectl on Linux
1. Download latest stable `kubectl` binary, make it executable, and move it to the `PATH`:
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
To download and setup a specific version of kubectl, run below command and replace `v1.14.1` with your desired version
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

### Installing kubectl on Mac
##### Can install kubectl either manually, or using Homebrew
1. To install manually, download the latest stable kubectl binary, make it executable, and movie it to `PATH` with following command:
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
To download and setup a specific version of kubectl, run below command and replace `v1.14.1` with your desired version
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
2. To install kubectl with Homebrew, simply run `brew install kubernetes-cli`


### Installing kubectl on Windows
1. Download binary directly or use `curl` from CLI... once downloaded, binaries need to be added to the `PATH`
- v1.14.1 binary: https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/windows/amd64/kubectl.exe
- Latest stable release: https://storage.googleapis.com/kubernetes-release/release/stable.txt
2. If you have `curl` installed, you can run the below command from the CLI
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/windows/amd64/kubectl.exe
```

### kubectl Configuration
- To access Kubernetes cluster, kubectl client needs the master node endpoint and proper credentials to interact with the master node's API
- While starting Minikube, startup process creates a configuration file called `config` inside the `.kube` directory
--- `.kube` is often referred to as the `dot-kube-config` file, resides in user's home directory
- Config file has all connection details needed by kubectl
-- By default kubectl binary parses this file to find master node's connection endpoint, along with credentials
-- To see the connection details, can either see the content of `~/.kube/config` file or run `kubectl config view`
- Can get info about Minikube cluster by running `kubectl cluster-info`
- Can debug/diagnose cluster issues with `kubectl cluster-info dump`
- Access Kubernetes dashboard by running `minikube dashboard`, opening a new tab on our browser.
-- If browser doesn't open another tab or display the Dashboard as expected, verify the output in terminal
-- You can always copy and paste the link from the terminal into your browser
- Running `kubectl proxy` gets kubectl to authenticate with API on master node and makes Dashboard available on slightly different URL than one generated earlier, this time available on proxy port 8001.
1. Run `kubectl proxy` to receive `Starting to serve on 127.0.0.1:8001`
- This address returns a list of API endpoints we can then hit
- To get the Dashboard UI, visit `http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy/#!/overview?namespace=default`
- With `kubectl proxy` running in a terminal window, you can also run `curl http://localhost:8001/` to access the available API endpoints
- Can curl into any endpoint to get more information on it
- Need to have a way to authenticate requests to API server- can do this by providing a Bearer Token when issuing a curl, or by providing a set of keys and certificates
-- Bearer Token-- access token generated by authentication server (API on master) that is given back to client, allowing them to connect back to API without providing further details or resources
--- GET the token:
```
$ TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
```
---GET the API server endpoint
```
$ APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
```
---Confirm that APISERVER stored same IP as Kubernetes master IP by issuing following 2 commands and comparing outputs
```
$ echo $APISERVER
https://192.168.99.100:8443

$kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
```
---Accesing API server using `curl` command as shown:
```
$ curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure
{
 "paths": [
   "/api",
   "/api/v1",
   "/apis",
   "/apis/apps",
   ......
   ......
   "/logs",
   "/metrics",
   "/openapi/v2",
   "/version"
 ]
}
```
-- Can also extract client certificate, client key, and certificate authority data from `.kube/config` instead of access token
--- Once extracted, they are encoded and passed with a `curl` command, such as:
```
$ curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca
```


## Chapter 7: Kubernetes Building Blocks and Object Model
- Kubernetes has an object model that represents different persistent entities in the cluster... the entitites describe 3 things.
1. What containerized apps are running and on which node
2. Application resource consumption
3. Different policies attached to apps, like restart/upgrade policies,  or fault tolerances, etc.

- Each object declares our intent/desired state under the `spec` section
- Kubernetes manages `status` section for objects, which records the actual state of the object
- Kubernetes Control Plane tries to match object's actual state to the desired state
- Examples of Kubernetes objects are Pods, ReplicaSets, Deployments, Namespaces
- When object is created, its configuration data section from below the `spec` field has to be submitted to the Kubernetes API server
- `spec` section describes desired state and basic info such as object name
- While API accepts `JSON` format, most often we provide `YAML` files that get converted to JSON by `kubectl` before being sent to the API
- `apiversion` is first required field and specifies the API endpoint on the API server we want to conect to
--It must match an existing version for the object type defined
- `kind` is second required field and specifies object type, in below example is `Deployment` but can also be `Pod`, `Replicaset`, `Namespace`, `Service`, etc.
- `metadata` comes third, holding object's basic info such as name, labels, namespace
- `spec` is the 4th required field and marks the beginning of the block defining the desired state of the Deployment in our object
--In the example, we want 3 Pods running at any given time
--Pods are created using Pods Template defined in `spec.template`
--Nested object, like a Pod, retains `metadata` and `spec` and lose the `apiVersion` and `kind`, both replaced by `templace`
- `spec.template.spec` defines desired state of the Pod-- our example Pod creates a single container running on `ngingx:1.15.11` image
- In the example below, we have two `spec` fields (`spec` and `spec.template.spec`)
- Example of Deployment object's configuration in `YAML` format:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
```
- Once deployment object is created, Kubernetes attaches the `status` field to the object

- Pods are the smallest/simplest Kubernetes object-- unit of deployment which represents a single instance of an application
- Pod is a collection of one or more containers that are schedule together on the same host with the Pod, share the same network namespace, and have access to mount the same external storage
- Pods don't have the ability to self-heal by themselves, which is why they're used with controllers that handle Pods' replication, fault tolerance, self-healing, etc.
- Controller examples include Deployments, ReplicaSets, ReplicationControllers, etc.
- We attach a nested Pod's specs to a controller object using the Pod Template
- `apiVersion` again comes first, must specify v1 for Pod object definition
- `kind` comes next, specifying Pod object type
- `metadata` holds the name and label for object
- `spec` marks beginning of the block defining desired state of Pod, aka `PodSpec`
- Pod creates single container running `nginx:1.15.11` from Docker
- Pod example below
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```


- Labels: key-value pairs attached to Kubernetes objects
- Used to organize and select a subset of objects based on requirements... many objects have the same label(s)
- Controllers use Labels to logically group together decouple objects instead of using object name/ID
- If you have `app` and `env` Label keys, you could have 4 apps
1. app = FE env = dev
2. app = BE env = dev
3. app = FE env = QA
4. app = BE env = QA
- `env=dev` selects the two Pods on Dev enviornment
- `app=frontend` selects frontend Pods
- Can specify a particular Pod by calling its labels

- Controllers use Label Selectors to select a subset of objects... Kubernetes supports two types of Selectors
1. Equality-Based Selectors: allow filtering of objects based on Label keys and values
--Matching done using `=`/`==`, or `!=`
--IE `env==dev` or `env!==qa`
2. Set-Based Selectors: allow filtering based objects based on a set of values
-- Can use `in`, `notin` operators for Label values, and `exist`, `does not exist` for Label keys
--`env in (dev,qa)` selects objects where the Label is set to either dev or qa, with `!app` we find objects with no Label key `app`

- ReplicationCtronller- no longer recommended, it is a controller that ensures a specified number of replicas of a Pod are running at any given time
- If there are more Pods than desired, replication controller will terminate the extra Pods
- If there are fewer Pods than desired, then the replication controller creates more Pods to match desired count
- Dont deploy a Pod alone since it can re-start itself
- Default controller is a Deployment which configures ReplicaSet to manage Pods' lifecycle

- ReplicaSet is next-gen ReplicationController
- RS supports equality- and set-based selectors, whereas ReplicationControllers only support euqality-based selector
--THIS IS THE ONLY DIFFERENCE (currently)
- RS lets us scale the number of Pods running a specific container app image
- Scaling can be done manually or through the use of autoscaler
- If a Pod is forced to terminate, causing the current and desired state to be different, ReplicaSet will detect the difference and create an additional Pod so current == desired
- ReplicaSet can be used independently as Pod controllers, but only offer a limited set of features
- a set of complementary features are provided by Deployments, the recommended controllers for orchestration of Pods


- Deployments manage the creation, deletion, and updates of Pods-- automatically creates a ReplicaSet which creates the Pod
- Deployment manages ReplicaSets and Pods
- DeploymentController is part of master node's controller manager, ensures current and desired state are always equal
- Allows for app updates and downgrades via `rollouts` and `rollbacks`, and directly manages its ReplicaSets for scaling
--EXAMPLE: `Deployment` creates `ReplicaSet A`, which creates Pods A, B, and C.  Each Pod Template is configured to run `ngingx:1.7.9` container image
- We can change Pods Templates from Deployment, and update the container image from `nginx:1.7.9` to `nginx:1.9.1`
- `Deployment` triggers a new `ReplicaSet B` for the new container versioned `1.9.1`, and the association represents a new recorded state of the `Deployment, Revision 2`
- Deployment `rolling uopdate` handles the seamless transitions 3 pods from `ReplicaSet A` at build `1.7.9` to `ReplicaSet B` at build `1.9.1`
- `rolling update` triggered when the Pods Templates are updated for a deployment
- Operations such as scaling or labeling the deployment do not trigger a rolling update, and therefore do not change the Revision #
- Once `rolling update` completes, `Deployment` will  show ReplicaSets A and B, where A is scaled to 0 pods and B is scaled to 3 pods
- This is how `Deployment` records prior state config settings, as `Revisions`
- Once `ReplicaSet B` and its Pods versioned at `1.9.1`, `Deployment` actively starts managing them
- `Deployment` keeps prior configuration states saved as Revisions, which play a factor in the ability to `rollback` from the `Deployment`

- Namespaces: allow us to partition cluster into virtual sub-clusters... useful if multiple teams and users use the same Kubernetes cluster
- Names of resources/objects created inside a Namespace are unique, but not across Namespaces in the cluster
- Run `kubectl get namespaces` to get a list of Namespaces
- Kubernetes creates four default Namespaces
1. `kube-system`: contains the objects created by Kubernetes, mostly control plane agents
2. `kube-public`:  Namespace that is unsecure and readable by anyone-- used for purposes such as exposing public info about the cluster
3. `kube-node-lease`: The newest Namespace, it holds node lease objects used for node heartbeat data
--Good Practice is to cereate more Namespaces to virtualize the cluster for users and dev teams
4. `default`: contains objects and resources created by administators and developers
-- we connect to `default` by default (WOW!)
- Resource Quotas allow us to divide cluster resources within Namespaces

##### Rollback/Rollout Demo
1. Start MiniKube by running `minikube start`
2. Create deployment, in this case running 1.15-alpine: `kubectl create deployment mynginx --image=nginx:1.15-alpine`
3. Get deployments, ReplicaSets, pods ONLY for app of mynginx `kubectl get deploy,rs,po -l app=mynginx`
4. Scale Kube for up to 3 Replicas: `kubectl scale deploy mynginx --replicas=3` -- note the ReplicaSet name (last 4-5 chars of name)
5. Can get Image by running  `kubectl describe deploy mynginx`
6. `kubectl rollout history deploy mynginx` shows Rollout histry
7. To display details of a revision run `kubectl rollout history deploy mynginx --revision=1`
-- Will get error if you try to get details of a revision that doesn't exist
8. To perform a rolling upgrade, in this case updating the image, run: `kubectl set image deployment mynginx nginx=nginx:1.16-alpine`
9. To rollback from Revision 2 to Revision 1, ru: `kubectl rollout undo deployment mynginx --to-revision=1`
-- This causes Revision 1 to become Revision 3, as it is the most recent change



## Chapter 8: Authentication, Authorization, Admission Control
- Every API request reaching the API server goes through 3 stages before being accepted by server and acted upon: Authentication, Authorization, and Admission Control
1. Authentication- logs a user in
- Kubernetes doesn't have an object callled user, and it doesn't store usernames or other related details in its store
- Even without that, Kubernetes can use usernames for access control and request logging
- Kubernetes can support anonymous requests if configured properly, along with requests from normal users and service accounts
- User Impersonation is also supported for a user to be able to act as another user, useful for admins when troubleshooting authorization policies
- Kubernetes has 2 kinds of users
1. Normal Users: managed outside of Kubernetes cluster via independent services like User/Client certificates, a file listing usernames/passwords, Google Accounts, etc.
2. Service Accounts: in-cluser processes communicate with API server to perform different operations
-- Most Service Accounts are created automatically via the API server, but can also be created manually
-- SA users are tied to a given Namespace and mount the respective credentials to communicate with API server as Secrets

##### Kubernetes uses different authentication moules
- Client Certificates: pass `--client-ca-file=SOMEFILE` option to API server to enable client certificate authentication
-- Need to reference a file containing one more more certificate authorities
- Static Token File: pass a file with a pre-defined bearer token with `--token-auth-file=SOMEFILE`
-- Tokens last indefinitely and cannot be changed without restarting the API server
- Bootstrap Tokens: currently in beta status, used mostly for bootstrapping a new Kubernetes cluster
- Static Password File: similar to Static Token File, can pass a file with basic auth details with the `--basic-auth-file=SOMEFILE` option
-- These credentials last indefinitely and cannot be changed without restarting the API server
- Service Account Tokens: automatically enabled authenticator that uses signed bearer tokens to verify requests
-- Tokens get attached to Pods using the ServiceAccount Admission Controller-- this allows in-cluster processes to talk to API server
- OpenID Connect Tokens: helps us connect with OAuth2 providers such as Azure AD, Salesforce, Google, etc.
- Webhook Token Authentication: verification of bearer tokens can be offloaded to a remote service
- Authenticating Proxy: lets us progra additional authentication logic
- Can enable multiple authenticators, and first module to successfully authenticate the request short-circuits the evaluation
- You should enable at least two methods: service account tokens authenticator and one of the user authenticators

2. Authorization- authorizes API requests added by the logged-in user
- User can send API requests to different operations after they successfully authenticate
- Those requests get authorized by Kubernetes using various modules
- More than one module can be configured per cluster, each module check in sequence
- Some of the API request attributes that are reviewed by Kubernetes include the user, group, extra, Resource, or Namespace 
- Attributes are evaluated against policies-- if successful, then request is allowed, otherwise it's denied
- If authorizer approves/denies request, that decision is immediately returned

##### Authorization Modules
- Node Authorizer: special-purpose mode that specifically authorizes API requests made by kubelets-- authorizes kubelet's read operations for services, endpoints, nodes, etc., and writes operations fo rpods, nodes, evetns, and more
- Attribute-Based Access Control (ABAC) Authorizer: Kubernetes grants access to API requests which combine policies with attributes.
-- To enable ABAC, need to start API server with the `--authorization-mode=ABAC` option
-- Also need to specify authorization policy with `--authorization-policy-file=PolicyFile.json`
-- In below example, user `student` can only read Pods in Namespace `1fs158`
```
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "student",
    "namespace": "1fs158",
    "resource": "pods",
    "readonly": true
  }
}
```
- Webhook Authorizer: Kubernetes offers authorization decisions to third-party services which return true if successful or false if failed
-- To enable Webhook authorizer, we need to start the API server with `--authorization-webhook-config-file=SOME_FILENAME` option, where `SOME_FILENAME` represents the configuration of the remote authorization service
- Role-Based Access Control (RBAC) Authorizer: can regulate access to resources based on roles of individual users... can restrict resource access by specific operations when creating the roles
- RBAC can dynamically configure policies
- To enable RBAC, start the server with the `--authorization-mode=RBAC` flag
-- These operations include `post`, `get`, `update`, `patch`, etc.
-- Two kinds of roles in RBAC, Role and ClusterRole
--- Role can be granted access to resources within a specific Namespace
---  ClusterRole can be granted same persmissions as Role but its scope is cluster wide
--Role example below:
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: lfs158
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
--`pod-reader` role assigned in example above, which only has access to read the Pods of `1fs158` Namespace
-- Once role is created can bind users with RoleBinding
-- RoleBinding: allows us to bind users in same Namespace as Role-- could also refer a ClusterRole in RoleBinding, which gives permissiosn to Namespace resources defined in ClusterRole within RoleBinding's Namespace
-- Rolebinding Exmaple where students user is granted access to read Pods of `1fs158` Namespace:
```
kind: RoleBinding
apiVersion: rbac.authorization/k8s.io/v1
metadata:
  name: pod-read-access
  namespace: 1fs158
subjects:
- kind: User
  name: student
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
-- ClusterRoleBinding: lets usg rant access to resources at a cluster-level and to all Namespaces

3. Admission Control- software modules that can modify/reject requests baed on some additional checks, like a pre-set quota
- Used to specifiy granular access policies, including allowing privileged containers, checking resource quota, etc
- Policies are forced using different admission controllers, such as ResourceQuota, DefaultStorageClass, AlwaysPullImages, etc.
-- These only come into effect after API requests are authenticated and authorized
- Start Kubernetes API with `--enable-admission-plugins` flag to use admission controls, and send a comma-delimited list of controller names
-- IE: `--enable-admission-plugins=NamespaceLifecycle,ResourceQuota,PodSecurityPolicy,DefaultStorageClass`



##### Authentication and Authorization Exercise
- Exercise assumes environment uses certificate and key from `/var/lib/minikube/certs/` and `RBAC` mode for authorization
1. Start Minikube by running `minikube start`
2. View content of client's config file by running `kubectl config view`, observing the only context `minikube` and only user, also `minikube`, created by default
3. Create `lfs158` Namespace by running `kubectl create namespace lfs158`
4. Make `rbac` directory and cd into it by running `mkdir rbac && cdrbac`
5. Create `private key` for `student` user with `openssl` tool, then create a `certificate signing request` for `student` with `openssl`
- First run `openssl genrsa -out student.key 2048` to generate private key
- Next create a certificate signing request for student by running `openssl req -new -key student.key -out student.csr -subj "/CN=student/O=learner"`
6. Next we create the YAML config file for the certificate signing request object, leaving `request` blank at first.
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
	name: student-csr
spec:
	groups:
	-	system:authenticated
	request:
	usages:
	-	digital signature
	-	key encipherment
	-	client auth
```
7. View the certificate, encode it in `base64`, and assign it to the request field by running `cat student.csr | base64 | tr -d '\n'` and then running `vim signing-request.yaml` to update the request field with the value generated from teh command above
- Make sure lines are space indented and not tab indented
8. Create the certificate signing object by running `kubectl create -f signing-request.yaml`-- bases the the request off the metadata name
9. List the certificate signing objects by running `kubectl get csr`, shows your requests and their status (initial status is pending)
10. Approve certificate by running `kubectl certificate approve student-csr`, where you specify the CSR we approve
11. Extract approved certifiace from certificate signing request by running `kubectl get csr student-csr -o jsonpath='{.status.certificate}' | base64 --decode > student.crt` then `cat student.crt`
12. Configure student user's credentials by assigning the key and certificate
`kubectl config set-credentials student --client-certificate=student.crt --client-key=student.key`
- View the certificate in the newly created certificate file
13. Create new context entry in kubectl client config file for `student` user associated for `lfs158` namespace in `minikube` cluster-- `kubectl config set-context student-context --cluster=minikube --namespace=lfs158 --user=student`
14. Create new `deployment` in same namespace while in the default`minikube context` by running `kubectl -n lfs158 create deployment nginx --image=nginx:alpine`
15. List pods from `context student-context` by running `kubectl --context=student-context get pods` to get the pods-- should fail first time b/c user has no permissions configured for `student-context`
- Steps 16- are to correct the above error
16. To fix this, first create a YAML config file for a `pod-reader` role object which only allows the `get`, `watch`, `list` actions in the namespace against pod objects... create `roles.yaml` and pass below:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reacher
  namespace: lfs158
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
17. Create the role by running `kubectl create -f role.yaml`, and view roles by running `kubectl -n lfs158 get roles`
18. Create config file for rolebinding object (rolebinding.yaml), to assign `pod-reader` role to `student` user... `rolebinding.yaml` below:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-read-access
  namespace: lfs158
subjects:
- kind: User
  name: student
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
19. Create rolebinding object by running `kubectl create -f rolebinding.yaml` and list it from the `lfs158` namespace with `kubectl -n lfs158 get rolesbindings`
20. Permissions have now been created and assigned, so we can list the pods running `kubectl --context=student-context get pods`

## Chapter 9: Services
- User needs to connect to Pods to access application, and each Pod has its own IP address
- If a Pod terminates and a new one is created, the new Pod has a new IP which won't be known to the outside world
- Service groups Pods and defines a policy to access them, resolving the changing IP issue
-- Grouping is achieved via Labels and Selectors
- If we have `app` label key with `frontend` and `db` as the values for different Pods... using selectors `app==frontend` and `app==db`, we group Pods into two sets, one with 3 Pods and one with 1 (with 3 `app==fe` selectors and 1 `app==db`)
- Assign a name to logical grouping, such as `frontend-svc` and `db-svc`
- Service object example below: create `frontend-svc` service by selecting all pods that have Label key of `=app` with the value of `frontend`... each Service IP is routable within the cluster, known as ClusterIP
-- ie-- 172.17.04 and 172.17.0.5 are Cluster IPs assigned to `frontend-svc` and `db-svc` Services
```
kind: Service
apiVersion: v1
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports
  - protocol: TCP
    port: 80
    targetPort: 5000
```

- Accessing Pods using Service Object: client connects to Service via `ClusterIP`, which forwards traffic to one of the attached Pods
- Service provides load balancing by default while selecting Pods for traffic forwarding
- We can call `targetPort` on Pod which receives traffic while Service is forwarding it
- In above example, `frontend-svc` requires requests from client on `port 80` and forwards these requests to one of the attached Pods on `targetPort 5000`
-- If `targetPort` is not defined then traffic will be forwarded to Pods on port where Service receives traffic
- Logical set of Pod's IP address with `targetPort` is referred to as `Service endpoint`... in the example, `frontend-svc` has 3 endpoints: 10.0.1.3:5000, 10.0.1.4:5000, and 10.0.1.5:5000
- Endpoints are created and managed automatically by Service and not by Kube cluster admin

- All worker nodes run a daemon called `kube-proxy`
- `kube-proxy`: watches API server on master node for addition/removal of Services and endpoints
- For each new service on each node, `kube-proxy` configures `iptables` rules to capture traffic for ClusterIP, and forwards it to one of the endpoints
- By doing this, any node can receive external traffic then route it internally within the cluster based on the `iptables` rules
- When service is removed, `kube-proxy` removes corresponding `iptables` rules on all nodes

##### Service Discovery: Two Methods to Discover Nodes
1. Environment Variables: as Pod starts on any worker node, the `kubelet` daemon running on that node adds a set of env vars in the Pod for all active services
- If we have a Service called `redis-master`, which exposes port 6379 and its clusterIP of 172.17.0.6, then shows the env vars on a newly created Pod
```
REDIS_MASTER_SERVICE_HOST=172.17.0.6
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://172.17.0.6:6379
REDIS_MASTER_PORT_6379_TCP=tcp://172.17.0.6:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=172.17.0.6
```
- By doing this, we need to be careful while ordering Services, as Pods will not have env variables set for Services that are created AFTER the Pods are created
2. DNS: Kubernetes has an addon that creates a DNS record for each Service using a `my-svc.my-namespace.svc.cluster.local` format
- Services within the same Namespace find other Services by their name-- if we add a Service `redis-master` in `my-ns` Namespace, all Pods in same Namespace lookup the Service by its name-- `redis-master
- Pods from other Namespaces lookup the same Service by adding the respective namespace as a suffix, such as `redis-master.my-ns`
- DNS is the more common/recommended solution

- While defining a Service need to choose its access scope.  Need to decide if the service...
-- Is ONLY accessible within the cluster
-- is accessible from within the cluster and the external world
-- Maps to an entity that resides inside or outside the cluster
- Access scope is decided by ServiceType, which can be configured when creating the Service

- ClusterIP: the default ServiceType-- represents a virtual IP that is received by a Service
-- Virtual IP is used to communicate with Service and is only accessible within the cluster
- `NodePort`: used with ClusterIP, it is a high-port dynamically picked value from the default range `30000-32767` that is mapped to the respectiveService from all worker nodes
-- IE: if mapped NodePort is 32233 for `frontend-svc`, then when we connect to any worker node on port 32233, the node would redirect all traffic to the assigned ClusterIP of 172.17.0.4
- Can also assign high-port number to NodePort from default range if preferred
- NodePort is useful when Services need to be accessible from external world... EU connects to any worker node on the specified high-port, which proxies request internally to ClusterIP of the Service.... request then forwarded to apps running in the cluster
- To access multiple applications from the external world, admins can configure a reverse proxy... an ingress-- and define rules that target Services within the cluster

- LoadBalancer ServiceType: NodePort and ClusterIP are automatically created, and external load balancer will route to them
- The Service is exposed at a static port on each worker node-- also exposed externally using underlying cloud provider's load balancer feature
- Will only work if infrastructure supports automatic creation of Load Balancers and have support in Kubernetes-- as is the case with AWS and Google Cloud Platform
- Without this configuration, LoadBalancer IP address field isn't populated and the Service will work the same as a NodePort service

- Service can be mapped to ExternalIP if it can route to one or more of the worker nodes
- Traffic that's ingressed into cluster with ExternalIP as the destination IP, gets routed to one of the Service endpoints
- This type of service needs an external cloud provider such as GCP or AWS
- ExternalIPs aren't managed by Kubernetes-- cluster admin has to configure routing that maps ExternalIP address to one of the nodes

- ExternalName: ServiceType that has no selectors and doesn't define any endpoints
- Accessed within the cluster, returns a `CNAME` record of an externally configured Service
- ExternalName is primarily used to make externally configured services available to apps within the cluster, such as `my-database.example.com`
- If externally defined Service resides within the same Namespace, you would just use `my-database`


## Chapter 10: Deploying Standalone Apps

##### Deploying Application with Dashboard
1. Start minikube and check status with `minikube start` and `minikube status`, if it looks good then launch dashboard UI with `minikube dashboard`
- Dashboard is connected to `default` Namespace by default, so all operations are in that Namespace
2. From dashboard, click `+CREATE` in the top right corner
- This is where we can create an application using a valid YAML/JSON config file, or manually from the `CREATE AN APP` section
3. Click `CREATE AN APP` and set app name to `webserver`, set Docker image to use `nginx:alpine` where `alpine` is the image tag, set Pods to 3, and No Servce
- Can click Show Advanced Options to specify options such as Labels, Namespace, Environment Variables, and more
- `app` Label is set by default to application name
4. Click on `Deploy` button to deploy `webserver` and create a ReplicaSet of `webserver-5b4b4fb996`, and 3 Pods with `webserver-5b4b4fb996-xxxxx`
- Add full URL from Container Image field `docker.io/library/nginx:alpine` if any issues with simple `nginx:alpine` image name
5. Run `kubectl get deployments` to list the deployments, `kubectl get replicasets` to list the ReplicaSets, and `kubectl get pods` to list all the Pods
6. Can run `kubectl describe pod <pod-name>` to see a pods description
- This command displays many details, but only need to focus on Labels field, which is `k8s-app=webserver` in this instace
7. Adding the `-L` option to `kubectl get pods` adds additional columns to output to list Pods with their label tag and k8s-app info.... ie `kubectl get pods -L k8s-app,label2`
8. We can also add `-l` option to select specific pods, in this case Pods that have `k8s-app` Label set to `webserver`: `kubectl get pods -l k8s-app=webserver`
9. Can run `kubectl delete deployments webserver` to delete the webserver Deployment-- this also deletes the ReplicaSet and Pods it generated

##### Deploying Application from CLI
1. Create webserver.yaml file looking like below example:
- Lines 844-850 represent Deployment, lines 851-855 represent ReplicaSet, and 856-864 represent Pods
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```
2. Run `kubectl create -f webserver.yaml` to deploy App, which creates ReplicaSet and Pods as defined in YAML config
3. Create `webserver-svc.yaml` file with NodePort ServiceType-- this will cause Kubernetes to open a static port on all worker nodes
- If we connect to that port from any node, we are proxied to the ClusterIP of the service
- Below is sample file
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```
4. Run `kubectl create -f webserver-svc.yaml` to create the service
5. Can also run `kubectl expose deployment webserver --name=web-service --type=NodePort` to Expose the Deployment
6. Run `kubectl get services` to get a list of the services, can run `kubectl describe services web-service` to get more details
- `web-service` uses` app=nginx` as a Selector to group our 3 pods which are listed as endpoints
- When a request reaches our Service, it will be served by one of the Pods listed in the Endpoints section
- Accessing App: pull the web-service port number from the `describe` command, and tack that onto the output of `minikube ip` in a web browser... can also run `minikube service web-service` to open in a browser


##### Liveness and Readiness Probes
- Containerized apps are scheduled to run in pods on nodes accross our cluster, and at times apps may become unresponsive or delayed
- Implementing Liveness and Readiness Probes allows `kubelet` to control the health of app running inside a Pod's container, and force a container restart for unresponsive apps
- During config, it's recommended to allow enough time for Readiness Probe to fail a few times before passing, and then checking Livness Probe
- If Readiness and Liveness Probes overlap, there could be risk that container never reaches ready state
- If container in Pod is running but app running in container isn't responding to requests, then container is useless
-- this can happen due to application deadlock or memory pressure--- recommended to restart container to make app available
- Can use Liveness Probe instead of restarting manually
-- Liveness proble checks app's health and if healthcheck fails, `kubelet` restarts affected container automatically
- Liveness Probes can be set by defining liveness command, liveness HTTP request, TCP liveness probe
- Below example checks existance of file `/tmp/healthy`:
- We check the existence of the file every 5 seconds using `periodSeconds`
- `initialDelaySeconds` requests the kubelet to wait 5 seconds before the first check
- the file gets deleted after 30 seconds before it is deleted, which triggers a kubelet restart
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

- Liveness HTTP Request-- kubelet sends HTTP GET request (to /healthz in below example) endpoint on port 8080
- If failure is returned, kubelet will restart affected container, otherwise container is considered to be alive
```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: X-Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds
```

- TCP Liveness Probe: kubelet attempts to open TCP socket to container that is running the app
- If successfull, then app is healthy, otherwise kubelet marks it as unhealthy and restarts the imapcted container
```
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

- Some apps need to meet criteria before they can serve traffic
- Conditions can include ensuring depending service is ready, acknowledging large dataset needing to be loaded, etc.\
- Readiness Probes can be used as wait for certain triggers to occur, and THEN server traffic from app
- Pod with containers that don't report ready status will not receive taffic from Kubernetes Services
- Readiness and Liveness Probes are configured similarly, so it remains the same
```
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```



## Chapter 11: Kubernetes Volume Management
- Volumes: directory backed by a storage medium-- storage medium, content, and access mode are determined by volume type
- Data stored in a container is deleted if it crashes, and `kubelet` will restart it with a clean start, which means it won't have any of the old data
-- Volumes corrects this issue and allows us to retrieve deleted data (check back on this)
- Directory mounted in a Pod is backed by the underlying Volume Type
- Volume Type decides properties of directory, such as size, content, default access mode etc... Volume Types include:
1. `emptyDir`: created for Pod as soon as it's scheduled on worker node-- volume's life is coupled with the Pod-- if Pod is deleted, so is `emptyDir` forever
2. `hostPath`: allows us to share a directory from the host to the Pod-- if Pod is terminated, content of Volume is still available on the host
3. `gcePersistentDisk`: can mount a Google Compute Engine(GCE) persistent disk into a Pod
4. `awsElasticBlockStore`: can mount AWS EBS Volume into a Pod
5. `azureDisk`: can mount an MSFT Azure Data Disk into a Pod
6. `azureFile`: can mount a MSFT Azure File Volume into a Pod
7. `cephfs`: an existing CephFS volume can be mounted into a Pod-- when Pod terminates, volume is unmounted and its contents are preserved
8. `nfs`: can mount an NFS share into a pod
9. `iscsi`: can mount an iSCSI into a Pod
10. `secret`: can pass sensitive info, such as passwords, to Pods
11. `configMap`: objects that provide config data, or shell commands and arguments, into a Pod
12. `persistentVolumeClaim`: can attach PersistentVolume to a pod


- PersistentVolume: subsystem that provides APIs to users and admins to manage and consume persistent storage
- it uses PerssitentVolume API resource type to manage Volume-- uses PersistentVolumeClaim API resource type to consume it
- In containerized world, storage is managed by storage/sys admins-- EU receives instructions to use storage
- PersistentVolumes helps us follow these rules accross the many VolumeTypes
- PV is network-attached storage in cluster, provisioned by admin
-- Can be dynamically provisioned based on StorageClass resource
- StorageClass contains pre-defined provisioners and parameters to create a PV... using PersistentVolumeClaims, user sends request for dynamic PV creation, which gets wired to StorageClass
- Some VolumeTypes that support managing storage using PV are GCEPersistentDisk, AWSElasticBlockStore, AzureFile, AzureDisk, CephFS, NFS, iSCSI

- PersistentVolumeClaim: request for storage by a user-- requested based on type, access mode, and size
- Three access modes are ReadWriteOnce(read-write by a single node), ReadOnlyMany(read-only by many nodes), and ReadWRiteMany (read-write many nodes)
- Once suitable PV is found, it is bound to a PVC... after successful bound, PVC resource can be used in a Pod
- Once user finishes work, attached PV can be released-- underlying PV can be reclaimed (for admin to verify/aggregate data), deleted (both data and volume are deleted), or recycled for future usage
- Container Storage Interface (CSI) makes installing enw CSI-compliant volume plugins easier


## Chapter 12: ConfigMaps
- ConfigMaps let us decouple configuration details from container image
- Pass config data as key-value pairs which are consumed by Pods or other components, as environment variables or volumes
- ConfigMaps can be created from literal values, config files, and from one or more files/directories
- Can create ConfigMap from literal values by running `kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2`
- Can display details for new ConfigMap by running `kubectl get configmaps my-config -o yaml`
-- the `-o yaml` option requests details to be generated in YAML format

- Can create configMap from a configuration file by generating YAML file (below) and running `kubectl create -f configmap-config-file.yaml`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology Pct. Ltd.
```

- Can create ConfigMap from a non-config file by first creating a file `permission-reset.properties` with the below data
```
kubectl create configmap permission-config1 --from-file=permission-reset.properties
```
- Then run `kubectl configmap permission-config --from-file=permission-reset.properties`

- We can retrieve key-value data of ConfigMap or specific ConfigMap keys as environment variables when we are inside a Container
- Below, `myapp-full-container` is a Container where environment variables receive values of the `fullconfig-map` ConfigMap keys
```
containers:
- name: myapp-full-container
  image: myapp
  envFrom:
  - configMapRef:
    name: full-config-map
```
- Below `myapp-specific-container`'s environment variables receive values form specific key-value pairs from separate ConfigMaps
- We get SPECIFIC_ENV_VAR1 environment variable to set value of SPECIFIC_DATA key from config-map-1 ConfigMap
- SPECIFIC_ENV_VAR2 environment variable set value of SPECIFIC_INFO key from config-map-2 ConfigMap
```
containers:
- name: myapp-specific-container
  image: myapp
  env:
  - name: SPECIFIC_ENV_VAR1
    valueFrom:
      configMapKeyRef:
        name: config-map-1
        key: SPECIFIC_DATA
  - name: SPECIFIC_ENV_VAR2
    valueFrom:
      configMapKeyRef:
        name: config-map-2
        key: SPECIFIC_INFO
```

- Can mount a `vol-config-map` ConfigMap as a Volume inside a Pod
- For each key in ConfigMap, a file gets created in the mount path (where file is named with key name), and content of that file becomes the key's value
```
containers:
- name: myapp-vol-container
  image: myapp
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
volumes:
- name: config-volume
  configMap:
    name: vol-config-map
```

- Secret: object that can be used to let us encode this information before we share it, for example
-- Frontend connects to backend using a password
-- During deployment creation, we can include BE password in Deployment's YAML file, but the password wouldn't be protected
-- Password would be exposed to anyone who can access the config file
- Can share info in the form of key-value pairs similar to ConfigMaps, so we can control how information in a Secret is used
- Secret object is referenced without exposing its content
- Secret data is stored as plain text inside `etcd`, so administrators must limit access to the API server and `etcd`
- Newer feature lets Secret data to be encrypted at rest while it is stored in `etcd`-- this feature needs to be enabled at the API server level
- To create Secret, run `kubectl create secret generic my-password --from-literal=password=mysqlpassword`
- Secret would be called `my-password` which ahs value of `password` key set to `mysqlpassword`
- After creating secret, can analyze it with `kubectl get secret my-password` and `kubectl describe secret my-password`

- Can manually create Secret from YAML config file
- Two types of maps for sensitive info inside a secret, data and stringData
-- `data` maps: each value of a sensitive information field must be encoded using base64
-- First create base64 encoding by running `echo mysqlpassword | base64`
-- Can decode by taking encoded value and running `echo "ENCODED-VALUE" | base64 --decode`
```
apiVersion: v1
kind: Secret
metadata:
    name: my-password
type: Opaque
data:
  password: bXlzcWxwYXNzd29yZAo=  
```
-- `stringData` maps: no need to encode value of sensitive information field-- value will be encoded when `my-password` Secret is created
```
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
stringData:
  password: mysqlpassword
```
- If this was thrown in a file named `mypass.yaml`, we could create the secret running `kubectl create -f mypass.yaml`

- To create a Secret from a file, first encode sensitive data by running `echo mysqlpassword | base64`, and then put it in a text file by running `echo -n 'BASE64-OUTPUT' > password.txt`
- Next create the secret from `password.txt` by running `kubectl create secret generic my-file-password --from-file=password.txt`
- After creating secret it can be analyzed with `kubectl get secret my-file-password` and `kubectl describe secret my-file-password`

- Secrets are consumed by Containers in Pods as mounted data volumes or environment variables, and referenced in their entirety or as specific key-values
- Using Secrets as Environment Variables:  Below references only `password` key of `my-password` Secret and assign its value to WORDPRESS_DB_PASSWORD environment variable
```
spec:
  container:
  - image: wordpress:4.7.3-apache
    name: wordpress
    env:
    - name: WORDPRESS_DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-password
          key: password
```

- Using Secrets as Files from a Pod: Can mount Secret as a Volume inside a Pod... below we create a file for each `my-password` Secret key (where files are named after the names of keys), the files containing values of Secret
```
spec:
  containers:
  - image: wordpresS:4.7.3-apache
    name: wordpress
    volumeMounts:
    - name: secret-volume
    mountPath: "/etc/secret-data"
    readOnly: true
volumes:
- name: secret-volume
  secret:
    secretName: my-password
```