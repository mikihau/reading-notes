## Kubernetes in Action

by Marko Luksa

### Part 1: Overview

#### Ch 1: Introducing Kubernetes
- A quick chapter introducing container technologies and why kubernetes comes in handy in deployment -- facing the challenges of the microservice architecture.
- The mechanisms for container isolation:
  - linux namespaces: like mnt, pid, UTS -- isolating resources for each container
  - linux control groups (`cgroups`): limiting the amount of systems resources each container can consume
- Some highlights of kubernetes: 
  - datacenters abstracted out as a single computational resource
  - declaritive deployment
  - dedicated scheduler provides better resource usage -- users don't worry about where the pods are deployed
  - monitoring & auto healing

#### Ch 2: First steps with Docker and Kubernetes
- Minikube: when installed locally, it runs a single node k8s cluster in a VM. Often used for local development.
- Some useful kubectl commands:
  - `cluster-info`
  - `get (node, pods) -o wide`
  - `describe node <xx>` (name of the node optional)
  - `run`
  - `expose <resource: po, svc, rc, deploy, rc> <resource_name>`
  - `scale rc <rc_name> --replicas=<num>`
- Pod: a group of tightly related containers that always gets deployed on the same logical host (worker node and linux namespace).
- ReplicationController: ensures the number of pod replicas are met by starting and terminating pods.
- Service: exposing multiple pods at a single static ip:port pair when pods come and go.
