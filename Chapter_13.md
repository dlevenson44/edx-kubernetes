# Chapter 13: Ingress
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
