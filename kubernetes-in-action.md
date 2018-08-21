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
