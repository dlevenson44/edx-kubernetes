# Chapter 2: Kubernetes
- Define Kubernetes
-- Kubernetes: open-source system for automating/managing deployment, scaling, and management of containerized applications
-- Kubernetes Case Studies: https://kubernetes.io/case-studies/

- Benefits of Kubernetes
1. Features mentioned in below section
2. Portable and extensible
3. Can be deployed on environment of our choice such as VM or bare metal or local
4. Modular and pluggable architecture
5. Can write custom APIs/plugins to extend functionalities
6. Thriving community with many commits to code and meetups


- Kubernetes Features
1. Automatical Binpacking: automatically schedules containers based on resource usage and constraints without disabling the containers
2. Self-healing: automatically replaces/reschedules the containers from failed nodes-- also kills/restarts containers that don't respond to health checks based on rules put in place
3. Horizontal Scaling: can automatically scale apps based on resource usage like CPU/RAM, can also support dynamic scaling based on customer metrics
4. Service Discovery and Load Balancing: groups sets of containers and refers to them via a DNS (Domain Name System)-- DNS is also called a service, and services can be discovered automatically
5. Automated rollouts/rollbacks without downtime
6. Secrets/configuration management: can manage secrets/configuration details for app without rebuilding the respective images, secrets let us share confidential data to app without exposing it to the stack configuration like on GitHub with .gitignore
7. Storage Orchestration: can automatically mount local, external, and storage solutions to containers in a seamless manner based on software-defined storage (SDS)
8. Batch Execution: execute multiple things at once


##### Evolution of Kubernetes
- Kubernetes is also referred to as k8s, 8 characters between k and s
- written in Go, inspired by Google Borg, donated by Google to CNCF
- Has new releases every 3 months generally 
- Kubernetes features/objects that can be traced to Borg are API servers, Pods, IP-per-Pod, Services, Labels


##### Explain what Cloud Native Computing Foundation (aka CNCF) does
- Hosted by the Linux Foundation, aims to accelerate the adoption of containers, microservices, and cloud-native applications
- Hosts a set of projects and provides resources to each project, even though each project operates independently
- For Kubernetes, provides neutral home for trademark and enforces proper use, provides license scanning, offers legal guidance on patent/copyright issues, creates open source training, curriculum, and certification, supports ad hoc activities, hosts activities and more