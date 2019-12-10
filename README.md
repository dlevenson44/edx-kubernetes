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
##### Chapter 9: Services

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