# Chapter 7: Kubernetes Building Blocks and Object Model
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
-- It must match an existing version for the object type defined
- `kind` is second required field and specifies object type, in below example is `Deployment` but can also be `Pod`, `Replicaset`, `Namespace`, `Service`, etc.
- `metadata` comes third, holding object's basic info such as name, labels, namespace
- `spec` is the 4th required field and marks the beginning of the block defining the desired state of the Deployment in our object
-- In the example, we want 3 Pods running at any given time
-- Pods are created using Pods Template defined in `spec.template`
-- Nested object, like a Pod, retains `metadata` and `spec` and lose the `apiVersion` and `kind`, both replaced by `templace`
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

