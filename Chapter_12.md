# Chapter 12: ConfigMaps
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
