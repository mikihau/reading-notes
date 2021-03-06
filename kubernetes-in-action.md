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
- Pod(`po`): a group of tightly related containers that always gets deployed on the same logical host (worker node and linux namespace).
- ReplicationController(`rc`): ensures the number of pod replicas are met by starting and terminating pods.
- Service(`svc`): exposing multiple pods at a single static ip:port pair when pods come and go.
- Some useful kubectl commands (prefix each with `kubectl`):
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
- Namespace(`ns`): split resources into multiple non-overlapping groups, and operate within one group at a time. Cluster-level resources like node is not tied to a namespace.
- Commands:
  - `explain pods(.<section>)`: lists (subsections of) attributes an object can contain
  - `create -f <filename> (-n <namespace>)`: create pod using the given manifest
  - `logs <pod_name> (-c <container_name>) (--previous)`: logs get auto rotated every day and 10MB; previous for log before restart
  - `port-forward <pod_name> <local_port>:<pod_port>`: talk to a pod for debug purposes `curl localhost:<local_port>`
  - `get po -L <label1,label2,...>`: show label values as separate columns
  - `label <resource> <name> <label>=<value> (--overwrite)`: set overwrite to change labels
  - `get po -l '(!)<label>(=<value>)',<more criteria>`: selecting labels -- also `!<value>`, `<label> (not)in (<val1>,<val2>)`
  - `annotate <obj> <obj_name> <org>/<annotation_key>=<value>`
  - `get <resource> -n <namespace> (--all-namespaces) (--watch)`: specify namespace and watch for modifications
  - `config set-context $(kubectl config current-context) --namespace(-n) <ns_name>`: switch to a new namespace
  - `delete po <pod1 pod2> (-l <label selectors>) (--all)`
  - `exec -it <pod> (-c <container>) bash` or `exec <pod> -- <command>`: '--' required if command has dashed args

#### Ch 4: Replication and other controllers: deploying managed pods
- The Kubelet on the node hosting a pod checks the liveness probe for the pod to decide if the pod is healthy. Use `describe` for restart details.
- A ReplicationController needs 3 things for its operations: number of replicas, a selector, and a template to bring up new instances. Changing the template does not affect existing pods.
- A pod is not tied to a rc because rc works by taking lable selections. But a pod's `metadata.ownerReferences` field contains rc info if its managed by an rc.
- ReplicaSet(`rs`): a ReplicaSet acts just like a ReplicationController, but newer and with more expressive pod selectors.
- DaemonSet(`ds`): a DaemonSet makes sure exactly 1 pod is running on each node (or a subset of nodes using a node selector).
- Job: a Job is a manager like ReplicaSet, but for short-lived jobs. Restart policy, number of completions, parallelism and time allowed for a pod to complete(`activeDeadlineSeconds`) can be configured.
- CronJob(`cj`): a CronJob is a job scheduled to run in the future. Make sure it's idempotent in case the 2 of the jobs get started at the same time.
- Commands:
  - `edit rc <name>`: opens up the yaml definition file. Change will be applied immediately upon saving the file.
  - `delete rc <name> --cascade=false`: deleting a rc without deleting the pods it manages

#### Ch 5: Services: enabling clients to discover and talk to pods
- Port can be named with pods, and services can reference these port names, so that when pod changes ports, services doesn't have to be changed also.
- A Service's cluster-ip can only be accessed from within the cluster (nodes, other pods etc). However other pods can discover services' IP and port from env vars (prefixed by the service name), or IP through internal dns (`<service-name>(.<namespace>)`).
- A Service can be headless if its cluster ip is set to None, and it will return all ready pod endpoints instead in a dns lookup (`nslookup <service_name>(.<namespace>)`).
- A Service can be created to connect to ip:ports outside the cluster, for pods to consume. This can be done by creating the service without a label selector, and manaully creating the Endpoints resource to the service. Alternatively it can be done by creating a `ExternalName` type of service, which is just a `CNAME` alias.
- To expose a service to external traffic (outside the cluster), can open a NodePort type of service that opens the same port of each node for external access. As an extension, a loadblancer can be set up fronting a single ip and rounting requests to different nodes. Client IP is not preserved to pods unless `externalTrafficPolicy: Local` is set up to disable that additional network hop.
- Ingress: a service can also be exposed through an Ingress resource (implementation depending on platform). An Ingress works on L7, and can be configured to route multiple domain names and paths to different services (DNS configuration needed on the client side), by looking up the service's endpoints and pass the request to a pod. It can also be configured to accept TLS traffic.
- Pods should have a readiness probe signaling whether it's ready to accept connections (at the start and during its lifetime). If a pod is not ready, it's not included in the service's endpoints.
- Commands:
  - `apply -f <file or url to file>`: the declarative way of configuring resources
  - `replace -f <file or url to file>`: the imperative way of editing resources; expects the resource to exit already
  - `run <pod-name> --image=<> --generator=run-pod/v1 --rm --restart=Never -- <command>`: run an ad hoc pod

#### Ch 6: Volumes: attaching disk storage to containers
- Volume: volumes is the way to share data among containers in the same pod. A volume is a subresource of pod, so it exists while the pod lives -- data might be able to get persisted depending on the volume's type.
  - Data for `emptyDir` and `gitRepo` volumes is lost when the pod terminates.
  - Data for `hostPath` volumes gets persisted on the node.
  - Can use external network-attached file systems for persistent data, which include storage services provided by cloud providers, nfs and so on.
- PersistentVolume(`pv`): a cluster-level resource (not namespaced). PV is a resource administered by cluster admins to decouple storage technology and app developers -- thus making the manifests universial to other K8s clusters. Access modes: ReadWriteOnce, ReadOnlyMany, ReadWriteMany (number of nodes).
-PersistentVolumeClaim(`pvc`): a developer can create a pvc, which bounds a pv by matching the required size and access mode of available pvs. Pods can then mount those pvcs. After pvcs are deleted, the underlying pvs can get recycled (data deleted and back to the pool), deleted, or retained (waiting for admin to take action).
- StorageClass(`sc`): another cluster level resource. a cluster might have different classes of storage with different characters and performance. A cluster admin can create scs, each attached to a provisioner, to auto provision pvs when pvcs come in. In the pvc, the dev can specify which sc the app wants to have.
- Use `storageClassName` to specify the sc. If key `storageClassName` doesn't exist, a default sc will be used. If `storageClassName` is set to an empty string, a pre-provisioned pv will be used.  

#### Ch 7: ConfigMaps and Secrets: configuring applications
- To pass command arguments in a docker container, use `ENTRYPOINT` to define the script, and `CMD` for the default args. The exec form is preferred (e.g. `ENTRYPOINT ["node", "app.js"]`) because the main process (PID=1) is the script, not the shell. In k8s container configs, use `command` and `args` to override the container's `ENTRYPOINT` and `CMD`.
- For env vars, define a `env` section for each container in a pod. Use `$(VAR)` to refer to other env vars in the value definition.
- To use values in cms as env vars, use either:
  - `pod.spec.containers.env.valueFrom.configMapKeyRef.name/key` to reference a single k/v pair
  - `pod.spec.containers.envFrom.configMapRef` to expose all k/v pairs as env vars, with an optional prefix
- ConfigMap(`cm`): cm entries can be exposed as `configMap` type of volume to get mounted into pod's containers. Can select a subset of entries (and where they're mounted to) by defining `configMap.items`. The `subPath` property can be used to unhide existing files in a dir when a file is mounted (but loses the ability to receive file updates).
- When mounting cm as a volume, the mounted files gets updated a while (~1min) after `kubectl edit cm <name>`.
  - For a pod, the volume gets updated atomically because the underlying implementation uses the symbolic link.
  - Not all pods receive the update at the same time, so there could be lags.
  - Files mount into existing dirs won't get the update.
- Secret: mostly same idea with cm, but when mounted, they're stored in memory and distributed carefully by k8s. It's recommended to mount the secret as a volume because env vars may get exposed unexpectedly. There are many types of them, including `generic`, `tls`, and `docker-registry`.
- Commands:
  - `create cm <name> --from-literal=<key>=<val> from-file(=<custom_keyname>)=<file/dir>`: create ConfigMap with strings, files and directories
  - `create secret <type> <name> --from-...`: create secret with specific type

#### Ch 8: Accessing pod metadata and other resources from applications
- Pod metadata can be exposed in 2 ways:
  - through env vars by defining env vars in `spec.containers.env.valueFrom.fieldRef.fieldPath` in the manifest, or `spec.containers.env.valueFrom.resourceFieldRef.resource` and `divisor` (for resource requests/limits). Labels and annotations can't be gotten through env vars because they change.
  - through the `downwardAPI` volume, allowing containers in the same pod to access other container's values. Container-level metadata (e.g.resources) needs to specify the container's name.
- The kubernetes API can be explored locally by proxying and then hit `http://localhost:8001`, or through the UI dashboard.
- 3 ways o talk to the API server from within a pod:
  1. Directly use the cert and token provided in the mounted secret volume at `/var/run/secrets/kubernetes.io/serviceaccount`:
    - `export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`: this verifies the API server is signed by certified authority
    - then `curl -H "Authorization: Bearer $TOKEN" https://kubernetes` to authenticate itself by setting the Auth header
  2. These steps can be delegated to an ambassador container running in the same pod (and proxying), so the main container can just access localhost:8001.
  3. By using a language specific client library.
- Commands:
  - `proxy`: runs a proxy server that accepts HTTP connections on local machine, and proxies that to the API server

#### Ch 9: Deployments: updating applications declaratively
- `Rolling-update` works by first adding additional label to pods, and changing the label selector of the rc, then creating a new rc, creating new pods of new versions, delete the old rc, and changing the pod labels back. This is bad because:
  - Changing labels for the user leads to confusion.
  - Under the hood commands are issued by the client so loss of connectivity may be an issue.
  - The command is imperative (not declarive).
- The Deployment(`deploy`) resource is a higher level resource than rs/rc, because it coordinates different rs-es during rollouts.
- Changing the pod template in the Deployment resources triggers a rollout, causing it to `RollingUpdate` (or `Recreate` by configurating the strategy) the rs and pods to a new version.
  - Modifying a pod template's ConfigMap won't trigger a rollout; to do it you need to create a new cm and have the template refer to this new cm.
  - After a rollout of a new version, the old rs is kept in case of an undo (can set `revisionHistoryLimit` for a deployment).
- There are 2 variables to controll the pace/rate of a rollout -- both can be numbers or percentages.
  - `maxSurge`: num of pods (rounds up) to exist above the desired replica count.
  - `maxUnavailable`: num of pods (rounds down) unavailable below the desired replica count (not below the number of pods for all created).
- A deployment can be paused and resumed halfway during a rollout to create a canary release (a portion of pods is new) or to delay a rollout after making multiple changes.
- Bad versions can be prevented by:
  - Setting a good `readinessProbe` for containers and a high `minReadySeconds` for the deployment, which says a container is considered ready only if probe is successful throughout this many seconds. So if the new pod fails during this window, the deployment is blocked because the new pod is not considered ready.
  - Setting a rollout deadline in case the rollout isn't making good progress -- doesn't auto abort though.
- Use 3 dashes in between resource definitions in a single yaml file.
- Commands:
  - `rolling-update <rc1> <rc2> --image=<>`: (outdated) command to update an app directly through rcs
  - `rollout status/history/pause/resume/undo deployment <name> (--to-revision=<version_num>)`: make an action on a higher level resource (deployments, daemonsets, statefulsets)
  - `patch <resource> <name> -p '{<json data>}'`: modify a single property without editing the file
  - `set image <resource> <name> <container_name>=<image_name>:<image_tag>`: modifies the image of any resource containing a container
  - `<command> --v=<level_number>`: increase the logging level of the command

#### Ch 10: StatefulSets: deploying replicated stateful applications
- `StatefulSet`: a StatefulSet maintains a set of pods so that each pod retains identity (name, hostname, network identity, other state) when it dies. Each pod can have it's own volumes.
  - pods addressed ordinally (pod-0, pod-1, pod-2 etc) -- when scaled down, the larger ones gets deleted
  - a headless service providing network identity for each pod for peer discovery (available in dns)
- Pods in a StatefulSet are boot up one at a time and killed one at a time (no scaling down when 1 pod unhealthy) because some stateful apps doesn't handle this well (e.g. 2 nodes storing replications get killed at the same time)
- When creating a StatefulSet, pvc templates go with pod templates so that each pod can get its own pvc. When pods are scaled down, the pvc are retained to prevent loss of data.
- We can also use a `List` kind of definition to specify multiple k8s resources in the same file.
- Ways to talk to a pod directly: execute curl commands in another pod, port forwarding, proxying and curl the api.
- Peer discovery can be done with a DNS SRV lookup to the headless services -- it'll return all pod addresses backing up the service.
- A statefulset gets a rollout in newer k8s versions. In older versions when the config file changes, old pods needs to be deleted manually for the change to take effect.
- When a node has network failure, k8s shows the pods there as unknown -- the pod might still be running, but the kubelet can't be contacted. After some time the pods gets forcibly evicted, but the origional pod can be back when the network issue goes away. Can forcibly delete a pod, but do that only when you know the node never comes back again.
- Commands:
  - `delete po <name> --force --grace-period 0`: forcibly deletes a pod without confirming the pod is actually running

### Part 3: Beyond the basics

#### Ch 11: Understanding Kubernetes internals
- A k8s cluster is split in 2 parts:
  - the Control Plane (on master node): etcd persistent storage, the API server, the scheduler, the controller manager
  - the worker nodes: the kubelet(the only component that can't be in a pod), kube-proxy, container runtime (docker etc)
  - add-ons: k8s dns server, dashboard, a ingress controller, heapster, container network interface (network plugin)
- etcd: key-value store used to persist manifests, cluster state and metadata. Can be more than 1 instance. Only talks to the API server.
  - Ensures data valid by optimistic locking and forcing all actions through the API server (also checks authorization).
  - Ensures consistency (multiple instances) by RAFT consensus algo so that majority can change the state.
- API server: CRUD interface to cluster state. Handles data validation and optimistic locking. `Kubectl` is a client. Requests go through configured plugins:
  - Authentication: plugins gets called in turn until the user (group) is decided.
  - Authorization: plugins decide if the user is authorized to do that operation.
  - Admission control: plugins modify the resource, e.g. adding labels, missing defaults etc -- used only when modifying state.
- A component or API client can request to watch a resource when it's changed -- changes are streamed by the API server.
- Scheduler: the scheduler assigns a node to each new pod, and change its metadata. The nodes are watching the change so they know as soon as it's scheduled. Can deploy multiple schedulers (pod needs to say what scheduler) or none (schedule manually). The default scheduling algorithm:
  - Filtering eligible nodes by node resource, pod preference (node name/label match), volume, taints, affinity, already taken host ports.
  - Then select the best node by some ranking policy.
- Controllers: reconcile the desired state and actual state by watching the states of resources, do stuff and update their status -- also periodically relisting info. They do stuff by talking to the API server to manipulate resources a level lower than them.
- Kubelet: responsible for registering the node resource when started, create and delete pods by the watch mechanism or a local file dir (running k8s components as pods), monitors pod states+events+resource consumptions and report them (probes, restarts).
-Kubernetes Service Proxy(kube-proxy): makes sure connections at node:port ends up in the backing pod (or a random pod if multiple are present). It has 2 modes: userspace proxy mode and iptables proxy mode.
- Add-ons
  - kube-dns: a service (as pod) whose ip is written to `/etc/resolv.conf` file of any deployed container.
  - Ingress: usually implemented as a reverse proxy, and watch services and endpoints -- proxies things directly into pods (not service ip) so that the ip of clients are preserved.
- Each pod has a infrastructural container containing linux namespaces.
- Communication between pods and pods and nodes need to be NAT-less (no network address translation). But the source ip becomes the node ip when packets go out of the cluster. Implementations of Container Network Interface (CNI) should be deployed as plugins into all cluster nodes.
- Services are handled by kube-proxies on nodes. When pod A send a packet to a service (ip:port), kube-proxy on pod A's node rewrites the ip:port pair to a randomly selected pod backing the service with the iptable rules -- service's ip:port is virtual so the ip itself doesn't mean anything. That's why a service can't be pinged.
- To make apps highly available, deploy multiple replicas, or set up leader-followers/active-passives.
- To make k8s itself highly available, deploy multiple masters and multiple instances of each of the 4 components in master. Etcd is designed as distributed, API server doesn't hold state (but needs to be fronted by an lb), scheduler and controller master needs to select a leader and run active-passive.
- Commands:
  - `get componentstatuses`: view the health of each controll plane component
  - `attach`: attaches to the main process of a container
  - `get events --watch`

#### Ch 12: Securing the Kubernetes API server
- Clients of the API includes both human users, managed by external mechanisms like sso, and applications running in pods.
- The authentication plugins are called in order until one of them returns the username, user id and group name (the rest of the plugins are skipped). The plugins determine the username etc by examining the client certificate, auth token (HTTP header), basic auth and others.
- ServiceAccount (`sa`): a namespaced resource for authentication used by pods. Each pod has 1 sa, but multiple pods can share (default to 1 sa per , token mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`). The default sa is called `<ns_name>:default`.
- Both users and sa's are associated with groups.
- Authorization plugins handle things an sa can or can't do. 
- Role: namespaced resource defining what verbs (get, list, create etc) can be used in the api.
- RoleBinding: namespaced resource associating a single Role with user(s), sa(s), group(s) -- can associate people outside the ns.
- ClusterRole: roles for resources not namespaced, and apis that doesn't represent resources (e.g. `/healthz`). It can also serve as a common role in all namespaces -- if its resource sections refers to any namespaced resources, it depends on whether the binding is RoleBinding or ClusterRoleBinding.
- ClusterRoleBinding: associates a ClusterRole with users/sa's/groups.
- K8s comes with a set of default ClusterRoles and ClusterRoleBindings. Some key ClusterRoles include `view`, `edit`, `admin` and `cluster-admin`.

#### Ch 13: Securing cluster nodes and the network
- A pod can use the host's namespace in a couple of ways: using host's network namespace(`spec.hostNetwork: true`), binding to host's port(`spec.containers.ports.hostPort: <port_num>`), and using host's PID(see processes running on the host)/IPC(communicate with other processes on the host) namespaces(`spec.hostPID/hostIPC: true`).
