# Kubernetes

These notes are from the EDX Kubernetes course, found here: https://www.edx.org/course/introduction-to-kubernetes.

This site was built using [GitHub Pages](https://pages.github.com/).

##### [Chapter 1: Container Orchestration](Chapter_01.md)
##### [Chapter 2: Defining and Discussing Kubernetes](Chapter_02.md)
##### [Chapter 3: Kubernetes Architecture](Chapter_03.md)
##### [Chapter 4: Installing Kubernetes](Chapter_04.md)
##### [Chapter 5: Minikube: Local Single-Node Kubernetes Cluster](Chapter_05.md)
##### [Chapter 6: Accessing Minikube](Chapter_06.md)
##### [Chapter 7: Kubernetes Object Model](Chapter_07.md)
##### [Chapter 8: Authentication, Authorization, Admission Control](Chapter_08.md)
##### [Chapter 9: Services](Chapter_09.md)
##### [Chapter 10: Deploying Standalone Apps](Chapter_10.md)
##### [Chapter 11: Kubernetes Volume Management](Chapter_11.md)

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


## Chapter 13: Ingress
- Routing rules are given a Service-- they exist as long as the Service does, and there are many rules because there are many Services in the cluster
- Ingress: decouples routing rules from application and centralize rules management, allowing us to update our app without worrying about its external access
- Ingress doesn't do any request forwarding by itself, it just takes definitions of traffic routing rules and implements them
- Ingress is fulfilled by Ingress Controller
- Ingress is a collection of rules that allow inbound connections to reach the cluster Service
- Ingress configures a Layer 7 HTTP/HTTPS load balancer for Services to allow inbound connections to reach the cluster Services
-- Above provides TLS (Transport Layer Security), Name-based virtual hosting, Fanout routing, Loadbalancing, and Custom rules
- With Ingress, users don't connect directly to service-- they reach the Ingress endpoint, and the request is forwarded from there to the desired Service
- In example below, user requests go to both `blue.example.com` and `green.example.com` go to the same Ingress endpoint... from there they are forwarded to `webserver-blue-svc` and `webserver-green-svc`
- Name-Based Virtual Hosting Ingress Rule Example:
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: virtual-host-ingress
  namespace: default
spec:
  rules:
  - host: blue.example.com
    http:
      paths:
      - backend:
          serviceName: webserver-blue-svc
          servicePort: 80
  - host: green.example.com
    http:
      paths:
      - backend:
          serviceName: webserver-green-svc
          servicePort: 80
```

- Example below is Fanout Ingress, where requests to `example.com/blue` and `example.com/green` are forwarded to `webserver-blue-svc` and `webserver-green-svc`
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: fan-out-ingress
  namespace: default
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /blue
        backend:
          serviceName: webserver-blue-svc
          servicePort: 80
      - path: /green
        backend:
          serviceName: webserver-green-svc
          servicePort: 80
```

- Ingress Controller: an app watching Master Node's API server for changes in Ingress resources and updates the Layer 7 Load Balancer as needed
- Kubernetes supports different Ingress Controllers-- which can be built on our own
-- GCE L7 Load Balancer Controller and Nginx Ingress Controller are common Ingress Controllers
-- Others include Istio, Kong, and Traefik
- Can enable Nginx Ingress Controller by running `minikube addons enable ingress`-- this is shipped with Minikube!
- Once Ingress Controller is deployed, we can create Ingress resource... if we use `virtual-host-ingress.yaml` with Name-Based Virtual Hosting, we can run `kubectl create -f virtual-host-ingress.yaml`
- Can now access `webserver-blue-svc` or `webserver-green-svc` after creating the Ingress resource, with each service using `blue.example.com` or `green.example.com`
- Need to update host config file `/etc/host` to Minikube IP for those URLs... after update, file should output:
```
127.0.0.1        localhost
::1              localhost
192.168.99.100   blue.example.com green.example.com 
```
- Can now open blue.example.com and green.example.com on browser and access each application



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