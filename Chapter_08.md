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
