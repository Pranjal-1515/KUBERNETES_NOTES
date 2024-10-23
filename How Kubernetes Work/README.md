# Kubernetes Architecture
Kubernetes cluster is split into two parts:

* The Kubernetes Control Plane
*  The (worker) nodes

![image](https://github.com/user-attachments/assets/56a74aa4-fff3-4eab-9088-8945d77d8a25)

## COMPONENTS OF THE CONTROL PLANE
The Control Plane is what controls and makes the whole cluster function. To refresh
your memory, the components that make up the Control Plane are:

* The etcd distributed persistent storage
* The API server
* The Scheduler
* The Controller Manager

These components store and manage the state of the cluster, but they aren’t what runs the application containers.

## COMPONENTS OF THE WORKER NODES
The task of running your containers is up to the components running on each worker node:

* The Kubelet
* The Kubernetes Service Proxy (kube-proxy)
* The Container Runtime (Docker, rkt, or others)

## ADD-ON COMPONENTS
Beside the Control Plane components and the components running on the nodes, a few add-on components are required for the cluster to provide extended services. This includes:

* The Kubernetes DNS server
* The Dashboard
* An Ingress controller
* Heapster
* The Container Network Interface network plugin

## HOW THESE COMPONENTS COMMUNICATE
Kubernetes system components communicate only with the API server. They don’t
talk to each other directly. The API server is the only component that communicates
with etcd. None of the other components communicate with etcd directly, but instead
modify the cluster state by talking to the API server.
 Connections between the API server and the other components are almost always
initiated by the components. But the API server does connect
to the Kubelet when you use kubectl to fetch logs, use kubectl attach to connect to
a running container, or use the kubectl port-forward command. <br> <br>
**NOTE:** The kubectl attach command is similar to kubectl exec, but it attaches
to the main process running in the container instead of running an additional one.

## HOW COMPONENTS ARE RUN
The Control Plane components, as well as kube-proxy, can either be deployed on the system directly or they can run as pods. You may be surprised to hear this, 
but it will all make sense later when we talk about the Kubelet. 
The Kubelet is the only component that always runs as a regular system component, and it’s the Kubelet that then runs all the other components as pods. To run the
Control Plane components as pods, the Kubelet is also deployed on the master. 

## How Kubernetes uses etcd
All the objects you’ve created throughout this book—Pods, ReplicationControllers, Services, Secrets, and so on—need to be stored somewhere in a persistent manner so
their manifests survive API server restarts and failures. For this, Kubernetes uses etcd, which is a fast, distributed, and consistent key-value store. Because it’s distributed,
you can run more than one etcd instance to provide both high availability and better performance. The only component that talks to etcd directly is the Kubernetes API server. All
other components read and write data to etcd indirectly through the API server. This brings a few benefits, among them a more robust optimistic locking system as well as
validation; and, by abstracting away the actual storage mechanism from all the other components, it’s much simpler to replace it in the future. It’s worth emphasizing that
etcd is the only place Kubernetes stores cluster state and metadata.

## ENSURING CONSISTENCY WHEN ETCD IS CLUSTERED
For ensuring high availability, you’ll usually run more than a single instance of etcd.
Multiple etcd instances will need to remain consistent. Such a distributed system
needs to reach a consensus on what the actual state is. etcd uses the RAFT consensus
algorithm to achieve this, which ensures that at any given moment, each node’s state is
either what the majority of the nodes agrees is the current state or is one of the previously agreed upon states. <br>
 Clients connecting to different nodes of an etcd cluster will either see the actual
current state or one of the states from the past (in Kubernetes, the only etcd client is
the API server, but there may be multiple instances). <br>
 The consensus algorithm requires a majority (or quorum) for the cluster to progress
to the next state. As a result, if the cluster splits into two disconnected groups of nodes,
the state in the two groups can never diverge, because to transition from the previous
state to the new one, there needs to be more than half of the nodes taking part in
the state change. If one group contains the majority of all nodes, the other one obviously doesn’t. The first group can modify the cluster state, whereas the other one can’t.
When the two groups reconnect, the second group can catch up with the state in the
first group.

## What the API server does
The Kubernetes API server is the central component used by all other components
and by clients, such as kubectl. It provides a CRUD (Create, Read, Update, Delete)
interface for querying and modifying the cluster state over a RESTful API. It stores
that state in etcd. <br>
 In addition to providing a consistent way of storing objects in etcd, it also performs
validation of those objects, so clients can’t store improperly configured objects (which
they could if they were writing to the store directly). Along with validation, it also handles optimistic locking, so changes to an object are never overridden by other clients
in the event of concurrent updates. <br>
 One of the API server’s clients is the command-line tool kubectl you’ve been
using from the beginning of the book. When creating a resource from a JSON file, for
example, kubectl posts the file’s contents to the API server through an HTTP POST
request.

## AUTHENTICATING THE CLIENT WITH AUTHENTICATION PLUGINS
First, the API server needs to authenticate the client sending the request. This is performed by one or more authentication plugins configured in the API server. The API
server calls these plugins in turn, until one of them determines who is sending the
request. It does this by inspecting the HTTP request. 
 Depending on the authentication method, the user can be extracted from the client’s certificate or an HTTP header, such as Authorization. 
 The plugin extracts the client’s username, user ID, and groups the user belongs to. This data is then used in the next stage, which is authorization.

## AUTHORIZING THE CLIENT WITH AUTHORIZATION PLUGINS
Besides authentication plugins, the API server is also configured to use one or more
authorization plugins. Their job is to determine whether the authenticated user can
perform the requested action on the requested resource. For example, when creating
pods, the API server consults all authorization plugins in turn, to determine whether
the user can create pods in the requested namespace. As soon as a plugin says the user
can perform the action, the API server progresses to the next stage.

## VALIDATING AND/OR MODIFYING THE RESOURCE IN THE REQUEST WITH ADMISSION CONTROL PLUGINS
If the request is trying to create, modify, or delete a resource, the request is sent
through Admission Control. Again, the server is configured with multiple Admission
Control plugins. These plugins can modify the resource for different reasons. They
may initialize fields missing from the resource specification to the configured default
values or even override them. They may even modify other related resources, which
aren’t in the request, and can also reject a request for whatever reason. The resource
passes through all Admission Control plugins. <br>
NOTE When the request is only trying to read data, the request doesn’t go
through the Admission Control.
Examples of Admission Control plugins include:

* AlwaysPullImages—Overrides the pod’s imagePullPolicy to Always, forcing
the image to be pulled every time the pod is deployed.
* ServiceAccount—Applies the default service account to pods that don’t specify
it explicitly.
* NamespaceLifecycle—Prevents creation of pods in namespaces that are in the
process of being deleted, as well as in non-existing namespaces.
* ResourceQuota—Ensures pods in a certain namespace only use as much CPU
and memory as has been allotted to the namespace.

## Understanding the Scheduler
The Kubernetes Scheduler is a core component of the Kubernetes control plane that is responsible for selecting which node an unscheduled pod should run on. It makes these decisions based on various factors, including resource requirements, constraints, and policies defined by the user.

## Introducing the controllers running in the Controller Manager
The API server doesn’t do anything except store resources in
etcd and notify clients about the change. The Scheduler only assigns a node to the
pod, so you need other active components to make sure the actual state of the system
converges toward the desired state, as specified in the resources deployed through the
API server. This work is done by controllers running inside the Controller Manager. 
 The single Controller Manager process currently combines a multitude of controllers performing various reconciliation tasks. Eventually those controllers will be split
up into separate processes, enabling you to replace each one with a custom implementation if necessary. The list of these controllers includes the:

* Replication Manager (a controller for ReplicationController resources)
* ReplicaSet, DaemonSet, and Job controller
* Deployment controller
* StatefulSet controller
* Node controller
* Service controller
* Endpoints controller
* Namespace controller
* PersistentVolume controller
* Others

## UNDERSTANDING THE KUBELET’S JOB
In a nutshell, the Kubelet is the component responsible for everything running on a
worker node. Its initial job is to register the node it’s running on by creating a Node
resource in the API server. Then it needs to continuously monitor the API server for
Pods that have been scheduled to the node, and start the pod’s containers. It does this
by telling the configured container runtime (which is Docker, CoreOS’ rkt, or something else) to run a container from a specific container image. The Kubelet then constantly monitors running containers and reports their status, events, and resource
consumption to the API server. 
 The Kubelet is also the component that runs the container liveness probes, restarting containers when the probes fail. Lastly, it terminates containers when their Pod is
deleted from the API server and notifies the server that the pod has terminated.

## Kubernetes Service Proxy
Beside the Kubelet, every worker node also runs the kube-proxy, whose purpose is to
make sure clients can connect to the services you define through the Kubernetes API.
The kube-proxy makes sure connections to the service IP and port end up at one of
the pods backing that service (or other, non-pod service endpoints). When a service is
backed by more than one pod, the proxy performs load balancing across those pods.

## HOW THE DNS SERVER WORKS
All the pods in the cluster are configured to use the cluster’s internal DNS server by
default. This allows pods to easily look up services by name or even the pod’s IP
addresses in the case of headless services.
 The DNS server pod is exposed through the kube-dns service, allowing the pod to
be moved around the cluster, like any other pod. The service’s IP address is specified
as the nameserver in the /etc/resolv.conf file inside every container deployed in the
cluster. The kube-dns pod uses the API server’s watch mechanism to observe changes
to Services and Endpoints and updates its DNS records with every change, allowing its
clients to always get (fairly) up-to-date DNS information. I say fairly because during
the time between the update of the Service or Endpoints resource and the time the
DNS pod receives the watch notification, the DNS records may be invalid.

## HOW (MOST) INGRESS CONTROLLERS WORK
Unlike the DNS add-on, you’ll find a few different implementations of Ingress controllers, but most of them work in the same way. An Ingress controller runs a reverse
proxy server (like Nginx, for example), and keeps it configured according to the
Ingress, Service, and Endpoints resources defined in the cluster. The controller thus
needs to observe those resources (again, through the watch mechanism) and change
the proxy server’s config every time one of them changes. 
 Although the Ingress resource’s definition points to a Service, Ingress controllers
forward traffic to the service’s pod directly instead of going through the service IP.
This affects the preservation of client IPs when external clients connect through the
Ingress controller, which makes them preferred over Services in certain use cases.
