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
- Pod: a group of tightly related containers that always gets deployed on the same logical host (worker node and linux namespace).
- ReplicationController: ensures the number of pod replicas are met by starting and terminating pods.
- Service: exposing multiple pods at a single static ip:port pair when pods come and go.
- Some useful kubectl commands:
  - `cluster-info`
  - `get (node, pods) -o wide (--show-labels) (-a)`: see more details (`-o yaml` for yaml config)
  - `describe node <xx>` (name of the node optional)
  - `run`
  - `expose <resource: po, svc, rc, deploy, rc> <resource_name>`
  - `scale rc <rc_name> --replicas=<num>`

### Part 2: Core Concepts

#### Ch 3: Pods: running containers in Kubernetes
- Best practice is to have one container per pod to facilitate finer grained scaling and management.
- Containers of the same pod share network interface (-> port space (so can communicate via localhost)), UTS namespace (-> hostname), but not file system. PID space can be configured to share.
- Behaving like physical hosts, each pod gets an IP address to communicate, regardless of which node it gets assigned onto.
- Label: arbitrary key-value pairs attached to k8s resources to organize them.
- Annotation: also key-value pairs to objects, but can't be grouped or selected
- Namespace: split resources into multiple non-overlapping groups, and operate within one group at a time. Cluster-level resources like node is not tied to a namespace.
- Commands:
  - `explain pods(.<section>)`: lists (subsections of) attributes an object can contain
  - `create -f <filename> (-n <namespace>)`: create pod using the given manifest
  - `logs <pod_name> (-c <container_name>) (--previous)`: logs get auto rotated every day and 10MB; previous for log before restart
  - `port-forward <pod_name> <local_port>:<pod_port>`: talk to a pod for debug purposes `curl localhost:<local_port>`
  - `get po -L <label1,label2,...>`: show label values as separate columns
  - `label <resource> <name> <label>=<value> (--overwrite)`: set overwrite to change labels
  - `get po -l '(!)<label>(=<value>)',<more criteria>`: selecting labels -- also `!<value>`, `<label> (not)in (<val1>,<val2>)`
  - `annotate <obj> <obj_name> <org>/<annotation_key>=<value>`
  - `get <resource> -n <namespace>`: specify namespace
  - `config set-context $(kubectl config current-context) --namespace <ns_name>`: switch to a new namespace
  - `delete po <pod1 pod2> (-l <label selectors>) (--all)`
  - `exec -it <pod> (-c <container>) -- bash` or `exec <pod> <command>`

#### Ch 4: Replication and other controllers: deploying managed pods
- The Kubelet on the node hosting a pod checks the liveness probe for the pod to decide if the pod is healthy. Use `describe` for restart details.
- A ReplicationController needs 3 things for its operations: number of replicas, a selector, and a template to bring up new instances. Changing the template does not affect existing pods.
- A pod is not tied to a rc because rc works by taking lable selections. But a pod's `metadata.ownerReferences` field contains rc info if its managed by an rc.
- ReplicaSet: a ReplicaSet acts just like a ReplicationController, but newer and with more expressive pod selectors.
- DaemonSet: a DaemonSet makes sure exactly 1 pod is running on each node (or a subset of nodes using a node selector).
- Job: a Job is a manager like ReplicaSet, but for short-lived jobs. Restart policy, number of completions, parallelism and time allowed for a pod to complete(`activeDeadlineSeconds`) can be configured.
- CronJob: a CronJob is a job scheduled to run in the future. Make sure it's idempotent in case the 2 of the jobs get started at the same time.
- Commands:
  - `edit rc <name>`: opens up the yaml definition file. Change will be applied immediately upon saving the file.
  - `delete rc <name> --cascade=false`: deleting a rc without deleting the pods it manages
