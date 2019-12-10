# Chapter 3: Kubernetes Architecture

##### Discuss Kubernetes Architecture
- At high level, Kubernetes has one or more master nodes, one or more worker nodes, and distributed key-value store, like etcd
- Master Node: responsible for managing Kubernetes cluster and is the entry point for all admin tasks
-- Can communicate with MN via CLI, GUI, or APIs
-- There can be multiple Master Nodes for fault tolerance purposes-- other MNs would be in a High Availability (HA) mode, with only one being the leader and performing all the operations
- Worker Node: Machine, VM or physical, which runs applications using Pods and is controlled by the MN
- Pod: Scheduling unit that is collection of one or more containers which are always scheduled together
- Pods are scheduled on the worker nodes, which have the tools to run and connect them
- We can access applications from external world via WNs and not MNs



##### Explain different components for master and worker nodes
###### Master Node has following components:
1. API Server: admin tasks performed via API server in MN, operator sends REST commands to API which processes/validates reqs
- After executing requests, resulting state of the cluster is stored in the ETCD
2. Scheduler: schedules work to different worker nodes using resource usage info for each worker node
- Schedules work in terms of Pods and Services
- Knows about contraints in place, such as scheduling work on a node that has the label `disk===ssd` set
- Before scheduling work, scheduler takes into account quality of service requirements, data locality, affinity, anti-afinity, etc.
3. Controller Manager: manages different non-terminating control loops, regulating the state of the cluster
- Each control loop knows about the desired state of the objects managed, and watches their current state through API server
- If current state of objects managed by control loop doesn't match desired state, control loop corrects to make sure current state === desired state
- `kube-controller-manager` runs controllers responsible to act when nodes become unavailable, to ensure pod counts are as expected, to create endpoints, service accounts, and API access tokens
- `cloud-controller-manager` runs controllers responsible for interacting with underlying infrastructure of a cloud provider when nodes are unavailable, to manage storage volumes when provided by cloud service, and to manage load balancing and routing
4. ETCD: Distributed key-value store that is used by Kubernetes to manage cluster state, and is connected to all Master Nodes
- Can be configured externally, in which case MNs would connect to it

###### Worker Node has following components:
1. Container Runtime: runs and manages container's lifecycle, some container runtimes are containerd, rkt, and lxd
2. Kubelet: Agent that runs on each WN and communicates with MN, receives Pod definition primarily through API, and runs containers associated with Pod
- Also makes sure that containers that are part of Pods are healthy at all times
- Connects to container runtime using Container runtime Interface (CRI)
-- CRI consistes of protocol buffers, gRPC API, and libraries
- Kubelet connects to CRI shim to perform container/image operations
- CRI implements ImageService and RuntimeService
-- ImageService repsonsible for image-related operations
-- RuntimeService responsible for Pod and container-related operations

3. Kubelet CRI Shims: dockershim- containers are created using Docker installed on WNs, internally, Docker uses containerd to create and manage containers
- cri-containerd: lets use use Docker's smaller offspring `containerd` to create/manage containers
- CRI-O: enables any Open Container Initiative (OCI) compatible runtimes with Kubernetes

4. Kube-proxy: network proxy running on each WN and listens to API server for each Service endpoint creation/deletion
- For each Servie endpoint, kube-proxy sets up the routes so that it can reach to it
- Service groups related Pods, and when accessed, load balances to them
-- Connect to Service isntead of to Pods directly


##### Discuss cluster state management with etcd
- etcd is based on Raft Consensus Algorithm, which allows a collection of machines to work as a coherent group that can survive failures of some members

- Networking Challenges-- microserves based apps rely on networking to mimic tight-coupling once available
1. Container-to-container communication inside Pods
2. Pod-to-Pod communication on the same node and across cluster nodes
3. Pod-to-Service communication within the same namespace and across cluster namespaces
4. External-to-Service communication for clients to access applications in a cluster

- Review Kubernetes Network setup requirements
1. Unique IP assigned to each Pod-- can be done with either Container Network Model (CNM, propposed by Docker) or Container Network Interface (CNI proposed by CoreOS).... Kubernetes uses CNI to assign IPs
- Container runtime offloads IP assignment to CNI, which connects to configured plugin such as Bridge or MACvlan, to get the IPs
- Once IP is given by plugin, CNI forwards it back to requested container runtime
2. Containers in Pod can communicate with each other
- Network Namespace: all container runtimes create an isolated network entity for each container that it starts
- Inside Pod, containers share network namespaces so they can reach to each other via localhost
3. Pod is able to communicate with other Pods in the cluster-- Pods can be scheduled on any node in a clustered environment
- We need to ensure that Pods can communicate across nodes and that all nodes should be able to reach any Pod
- Kubernetes puts condition that there shouldn't be any Network Address Translation(NAT) while Pod-toPod communication across hosts occurs
-- This is done via routable pods/nodes using the underlying physical infrastructure like Google Kubernetes Engine
-- OR done by using Software Defined Networking like Flannel, Weave, Calico, etc.
4. If configured, application deployed inside a Pod is accessible from the external world-- by exposing services to external world, we can access applications from outside the cluster

##### Communication Across Nodes
- Pods are scheduled on nodes randomly in a Kubernetes cluster
-- Regardless of host node, Pods are expected to communicate with all other pods in the cluster without implementing a Network Address Translation (NAT)
-- FUNDAMENTAL REQUIREMENT OF ANY NETWORK IMPLEMENTATION IN KUBERNETES
- Kubernetes network model aims to reduce complexity, treats Pods as VMs ona network where each VM receives an IP-- thus assigning each Pod an IP
- This model is called IP-per-Pod, ensures proper Pod-to-Pod communication
- Containers share Pod's network namespace and must coordinate ports assignment inside the Pod just as apps would on a VM
- Must do this while communicating with each other on localhost inside the Pod
- Containers are integrated with overall Kubernetes network model through use of Container Network Interface (CNI), supported by CNI plugins
-- CNI is a set of specification/libraries which lets plugins configure networking for containers
-- There are a few core plugins, most are 3rd party Software Defined Networking (SDN) solutions implementing the Kubernetes networking model
-- Some CNIs also offer support for Network Policies
-- Flannel, Weave, and Calico are a few SDN solutions available for Kubernetes clusters
- Container runtime offloads IP assignment to CNI which connects underlying configured plugin to get IP address
-- Once IP is given, CNI forwards it back to requested container runtime

##### Pod-to-External World Communication
- To deploy containerized apps running in Pods in a Kubernetes cluster, it needs accessibility from the outside world
- Kubernetes enables external accessibility through services
-- services: complex consutrcts which encapsulate networking rules definitions on cluster nodes
- By exposing services to the external world with kube-proxy, apps become accessible from outside the cluster over a virtual IP