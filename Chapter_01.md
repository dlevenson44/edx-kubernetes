# Chapter 1: Container Orchestration
##### Define concept of container orchestration
- Container orchestration groups multi-container applications together at scale

##### Explain reasons for container orchestration:
- Better performance and tools, such as binding containers and storage, keeping resource usage in-check and optimizing as needed, and scheduling containers to run ond ifferent hosts

##### Different container orchestration options and tools
- Amazon Elastic Container Service hosted by AWS to run Docker containers at scale
- Azure Container Instances hosted by Microsoft Azure is basic container orchestration service
- Azure Service Fabric is open source container orchestrator provided by Msft Azure
- Kubernetes is open source tool started by Google and is part of Cloud Native Computing Foundation
- Marathon is framework to run containers at scale on Apache Mesos
- Nomad is container orchestrator from HashiCorp
- Docker Swarm is container provider by Docker Inc, part of Docker Engine

##### Different container orchestration deployment options
- VMs, on-premise, baremetal, workstation, laptop, datacenter, AWS, Google Cloud, Azure Cloud


- Containers: Application-centric way to deliver high-performing, scalable apps on your choice of infrastructure
- Container Images: wrap application code, runtime, and dependencies in a  pre-defined format
- Container Runtimes: use the pre-packaged images to create one or more containers
-- Examples of Container Runtimes are runC, containerd, and rkt, are all good at running containers on a single host
- Container Orchestrator:  A single controller/management unit/tool that groups hosts together to form a cluster and help us meet the below requirements needed for a PROD environment:
-- Fault-tolerant, scalable, uses resources optimally, can automatically discover and communicate with other apps, accessible from the external world, can update/rollback without any downtime
-- Different container orchestrators include Docker Swarm, Kubernetes, Mesos Marathon, Amazon ECS, Hashicorp Nomad
--- Differences explored here:  https://www.edx.org/course/introduction-to-cloud-infrastructure-technologies
- Container Orchestrators make things easier by able to:
1. Bring multiple hosts together and make them part of a cluster
2. Schedule containers to run on different hosts
3. Help containers running on one host reach out to containers running on other hosts in the cluster
4. Bind containers and storage
5. Bind containers of similar type, such as services, to a higher-level construct so there are less individual containers to manage
6. Keep resource usage in-check adn optimize as needed
7. Allows secure access to apps running inside containers

- Container Orchestrators can be deployed on bare metal, VMs, on-premise, or on a cloud of our choice.
-- For example, it can be deployed on workstation/laptop, in a datacenter, on AWS, on OpenStack
-- One-click installers exist to set up Kubernetes on the cloud, such as Google Kubernetes Engine on Google Cloud, or Azure Container Service on MSFT Azure

- Container image bundles application, runtime, and dependencies, which is then used to create an isolated executable environment, aka a container
- Containers can be deployed froma  given image on the platform of our choice, such as desktops, clouds, VMs, etc.
