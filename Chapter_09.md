# Chapter 9: Services
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
