# Chapter 6: Accessing Minikube
- kubectl-- CLI tool to access Minikube Kubernetes cluster
-- Manages cluster resources and applications
-- Can be used standalone or as part of scripts and automation tools
-- Once all required credentials and cluster access points have be configured for kubectl, it can be used remotely from anywhere to access a cluster
-- Receives configuration from Minikube Kubernetes cluster access-- other cluster setups may need to have the acces points and certificates configured so it meets kubectl requirements
-- Recommended to keep `kubectl` at same version with the Kubernetes run by MiniKube
Kubernetes Dashboard-- web-based UI to interact with the cluster, manage resources and containerized apps
- curl-- commands you can write to access cluster via APIs with the proper credentials
-- Can connect to API using endpoints and send commands to it

##### HTTP API space of Kubernetes is divided into 3 groups:
1. Core Group (/api/v1)-- group includes objects such as Pods, Services, nodes, namespaces, configmaps, secrets, etc.
2. Named Group-- group includes objects in `/apis/$NAME/$VERSION` format-- different API versions imply different levels of stability and support
- Alpha: may be dropped at any point in time without notice, IE `/apis/batch/v2alpha1`
- Beta: well-tested, but semantics of objects may change in incompatible ways in a subsequent beta/stable release, IE `/apis/certificates.k8s.io/v1beta1`
- Stable: appears in released software for many subsequent versions, IE `/apis/networking.k8s.io/v1`
3. System-wide-- group consists of system-wide API endpoints such as `/healthz`, `/logs`, `/metrics`, `/ui`, etc.
- Can either connect to API directly calling teh API endpoints or through the CLI/Web UI


#### Installing kubectl on Linux
1. Download latest stable `kubectl` binary, make it executable, and move it to the `PATH`:
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
To download and setup a specific version of kubectl, run below command and replace `v1.14.1` with your desired version
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### Installing kubectl on Mac
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


#### Installing kubectl on Windows
1. Download binary directly or use `curl` from CLI... once downloaded, binaries need to be added to the `PATH`
- v1.14.1 binary: https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/windows/amd64/kubectl.exe
- Latest stable release: https://storage.googleapis.com/kubernetes-release/release/stable.txt
2. If you have `curl` installed, you can run the below command from the CLI
```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/windows/amd64/kubectl.exe
```

#### kubectl Configuration
- To access Kubernetes cluster, kubectl client needs the master node endpoint and proper credentials to interact with the master node's API
- While starting Minikube, startup process creates a configuration file called `config` inside the `.kube` directory
-- `.kube` is often referred to as the `dot-kube-config` file, resides in user's home directory
- Config file has all connection details needed by kubectl
- By default kubectl binary parses this file to find master node's connection endpoint, along with credentials
- To see the connection details, can either see the content of `~/.kube/config` file or run `kubectl config view`
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
- Bearer Token-- access token generated by authentication server (API on master) that is given back to client, allowing them to connect back to API without providing further details or resources
-- GET the token:
```
$ TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
```
---GET the API server endpoint
```
$ APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
```
-- Confirm that APISERVER stored same IP as Kubernetes master IP by issuing following 2 commands and comparing outputs
```
$ echo $APISERVER
https://192.168.99.100:8443

$kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
```
--- Accesing API server using `curl` command as shown:
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
- Can also extract client certificate, client key, and certificate authority data from `.kube/config` instead of access token
-- Once extracted, they are encoded and passed with a `curl` command, such as:
```
$ curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca
```
