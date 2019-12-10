
## Chapter 14: Advanced Topics

- Annotation: allows us to attach arbitrary non-identifying metadata to any objects in a key-value format
- Not used to ID/select objects like Labels do
- Used to store build/release IDs, PR numbers, git branch... pointers to logging/monitoring/analytics/debugging.... phone/pager numbers of people responsible
```
"annotations": {
  "key1": "value1",
  "key2": "value2"
}
```
- Can add Annotation description to Deployment on creation, example below:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver
  annotations:
    description: Deployment based PoC dates 2nd May 2019
```
- Can then display annotation by running `kubectl describe deployment webserver`


- Job: creates one or more Pods to perform a task-- Job object takes responsibility of Pod failures, makes sure the task is completed successfully
- Once task completes, all Pods terminate automatically
- Job configuration options include `parallelism` (set the number of pods to run in parallel), `completions` (set number of expected completions), `activeDeadlineSeconds` (set duration of the Job), `backoffLimit` (set number of retries before Job is marked as failed), and `ttlSecondsAfterFinished` (to delay cleanup of finished jobs)
- CronJobs: Jobs that are performed at scheduled dates/times
- CronJob options include `startingDeadlineSeconds` (set deadline to start job if scheduled time was missed), `concurrencyPolicy` (allows/forbids concurrent Jobs or replace old Jobs with new ones)

- ResourceQuota API: provides constraints that limit aggregate resource consumption on a per Namespace basis
- When there are many users sharing a Kubernetes cluster, there's a concern for fair usage-- all users should have same usage ability
- Can set 3 types of quotas per Namespace:
1. Compute Resource Quota: can limit the total sum of the compute resources (CPU, memory, etc) that can be requested in a given Namespace
2. Storage Resource Quota: can limit total sum of storage resources (PersistentVolumeClaims, requests.storage, etc.)that can be requested
3. Object Count Quota: can restrict number of objects a given type (Pods, ConfigMaps, ReplicationControllers, Services, Secrets, PErsistentVolumeClaims, etc.)

- Autoscaling: implemented in Kubernetes cluster via controllers that adjust the number of running objects based on single, multiple, or custom metrics
- Practical for production-ready cluster where hundreds or thousands of objects are deployed
- Is a dynamic scaling solution that adds/removes objects from cluster based on resource utilization, avialability, and requirements
- 3 main types of autoscalers that can be used individually or as a combination for autoscaling solutions
1. Horizontal Pod Autoscaler (HPA): algorithm based controller API resource that adjusts number of replicas in ReplicaSet, Deployment, or Replication Controller based on CPU utilization
2. Vertical Pod Autoscaler (VPA): sets Container resource requirements (CPU and memory) in a pod and adjusts the requirements in runtime, based on historical utilization data, current resource availability, and real-time events
3. Cluster Autoscaler: automatically re-sizes Kubernetes cluster when there are insufficient resources avialable for new Pods expecting to be schedueld, or when there are underutilized nodes in the cluster

- DaemonSet: specific Pod type that is running on all nodes at all times
- Allows us to collect monitoring data from all nodes or to run a storage daemon on all nodex
- Is a critical controller API for multi-node Kubernets clusters
- `kube-proxy` agent running as a Pod on every single node in cluster is managed by DaemonStat
- When a node is added to the cluster, Pod from a given DaemonSet is auotmatically created on that cluster
- DaemonSet's Pods are placed on nodes by the cluster's default Scheduler
- When node dies or is removed from cluster, Pods are garbage collected--- if DaemonSet is deleted, all Pods it created are deleted too
- DaemonSet resource allows its Pods to be scheduled only on specific nodes by running `nodeSelectors` and node `affinity` rules
- DaemonSets support rolling updates and rollbacks like Deployments

- StatefulSet: Controller used for stateful applications that require a unique ID, such as name/network.... for example `MySQL cluster, etcd cluster`
- StatefulSet controller provides identy and guaranteed ordering of deployment and scaling to Pods
- StatefulSets use ReplicaSets as intermediary Pod controllers and support rolling updates and rollbacks like Deployments

- Kubernetes Cluster Federation: lets us manage multiple Kubernetes clusters from a single control plane
- Can sync resources across federated clusters and have cross=cluster discovery
- Lets us perform Deployments across regions, access them using a global DNS record, and achieve High Availability
- Still in Alpha, Federation is useful when we want to build a solution where we can have one cluster running inside our private datacenter and another one is in the public cloud-- a hybrid solution
- Hybrid solution lets us avoid provider lock-in
- Can also assign weights for each cluster in the Federation to distribute load based on custom rules

- Resource: API Endpoint that stores a collection of API objects-- ie a Pod resource contains all the Pod objects
- Cu stom Resources: resources that don't have to modify the Kubernets source
- Dynamic in nature, can appear/disappear in an already running cluster at any time
- To make resource declarative, we need to create and install a custom controller that can interpret the resource structure, and performed the needed actions
- Custom contrllers can be deployed and managed in an already running cluster
- Two ways to add custom Resources
1. Custom Resource Definitions (CRDs): easiest way to add custom resource and doesn't require any programming knowledge... building custom controller WOULD require some programming
2. API Aggregation: More fine-grained control, subordinate API servers that sit behind primary API server
- Primary API acts as a proxy for all incoming API requests - handles ones based on its capabilities and proxies over requests meant for subordinate APIservers

- Kubernetes manifests such as Deployments, Services, Ingress, and Volume Claims deploy applications, but i tcan be tiresome deploying them one by one
- Chart: a bundle that manifests after templatizing them into a well-defined format with other metadata
- Charts can be server via repositories like a rpm or deb package
- Helm: Package manager (like `apt` for Linux) for Kubernetes that can install/update/delete Charts in the Kubernetes cluster
- Helm has two components-- helm which runs on user's workstation, and tiller which runs inside Kubernetes cluster
- Client helm connects to server tiller to manage Charts

- Security Contexts: allow us to set Discretionary Access Control for object access permissions, privileged running, capabilities,s ecurity labels, etc.
- Lets us define specific permissions for Pods and Containers
- Effect is limited to individual Pods and Containers where context configuration settings are incorporated into the `spec` section
- Pod Security Policies: allows us to apply security settings to multiple Pods and Containers cluster-wide
- Allows for more fine-grained security settings to control usage of host namespace, networking, and ports, file system groups, and more

- Network Policies: sets of rules that define how Pods can talk to other Pods and resources both inside and outside the cluster
- Pods not covered by a Network Policy will continue to receive unrestricted traffic from any endpoint
- Resitrcts communication between Pods and applications in cluster Namespace
- Similar to firewalls-- designed to protect mostly assets located inside Firewall but can restrict outgoing traffic as well
- Network Policy API resource specifies `podSelectors`, Ingress/Egress `policyTypes`, and rules based on source/destination `ipBlocks` and `ports`
- Recommended to define a default deny policy to block all traffic to and from NAmespace, then define sets of rules for specific traffic to be allowed in and out of Namespace
- Network Policies are namespaced API resources by default, and certain network plugins provide additional features so Network Policies are applied cluster-wide

- Metrics Server: cluster-wide aggregator of resource usage data by Pods, Services, Nodes, and more
- Prometheus: Part of CNCF, can be used to scrape resource usage from different Kubernetes components and objects
--Using its client libraries, can also isntrument code of our application
- Elasticsearch: most common way to collect logs, uses `fluentd` which is an open source data collector, also part of CNCF