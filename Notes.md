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



