# Chapter 10: Deploying Standalone Apps

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
