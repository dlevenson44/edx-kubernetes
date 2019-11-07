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

## Chapter 8: Services
