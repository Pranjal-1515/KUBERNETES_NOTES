# PODS
A pod is a group of one or more tightly related containers that will always run
together on the same worker node and in the same Linux namespace(s). Each pod
is like a separate logical machine with its own IP, hostname, processes, and so on,
running a single application. The application can be a single process, running in a
single container, or it can be a main application process and additional supporting
processes, each running in its own container. All the containers in a pod will appear
to be running on the same logical machine, whereas containers in other pods, even
if they’re running on the same worker node, will appear to be running on a different one.

## Pods in a Kubernetes cluster are used in two main ways:
**1.Pods that run a single container:** The "one-container-per-Pod" model is the most common Kubernetes use case; 
in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.

**2.Pods that run multiple containers that need to work together:** A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. 
These co-located containers form a single cohesive unit.

## Understanding what happened behind the scenes after ( kubectl run <name> --image=<image-name> --port=8080 ) Command
When you ran the kubectl command, it created a new Pod
object in the cluster by sending a REST HTTP request to the Kubernetes API server.
Then Pod scheduled to one
of the worker nodes by the Scheduler. The Kubelet on that node saw that the pod was
scheduled to it and instructed Docker to pull the specified image from the registry
because the image wasn’t available locally. After downloading the image, Docker created and ran the container.

## INTER-POD NETWORK
All pods in a Kubernetes cluster reside in a single flat, shared, network-address space, which means every pod can access every other pod at the other pod’s IP address. No NAT (Network Address Translation) gateways exist between them.
When two pods send network packets between each other, they’ll each see the actual IP address of the other as the source IP in the packet.
Consequently, communication between pods is always simple. It doesn’t matter if two
pods are scheduled onto a single or onto different worker nodes; in both cases the
containers inside those pods can communicate with each other across the flat NAT less network, much like computers on a local area network (LAN), regardless of the
actual inter-node network topology. Like a computer on a LAN, each pod gets its own
IP address and is accessible from all other pods through this network established specifically for pods.

## WHEN TO USE MULTIPLE CONTAINERS IN A POD
The main reason to put multiple containers into a single pod is when the application
consists of one main process and one or more complementary processes For example, the main container in a pod could be a web server that serves files from
a certain file directory, while an additional container (a sidecar container) periodically downloads content from an external source and stores it in the web server’s
directory.
 Other examples of sidecar containers include log rotators and collectors, data processors, communication adapters, and others.

 ## Creating pods from YAML
 ![cap](https://github.com/user-attachments/assets/937e53af-d428-44c9-9b69-a41c0998c2bb)

 **1. apiVersion**: Specifies the version of the Kubernetes API that this object is using. In this case, it's version 1 (v1), which is typically used for basic objects like 
 Pods.

 **2. kind**: Defines the type of Kubernetes object being described. Here, it indicates that the object is a Pod.

 **3. metadata**: This section provides information that helps identify the Pod.

 **4. name**: The name of the Pod. In this case, it’s called nginx. This name must be unique within the namespace.

 **5. spec**: This section defines the desired state and characteristics of the Pod.

 **6. containers**: Lists the containers that will run in the Pod. Each container is specified in a nested object.

 **7. name**: The name of the container within the Pod. This container is also named nginx.

 **8. image**: Specifies the container image to use. Here, it uses the nginx image with the version 1.14.2.

 **9. ports**: Defines the ports that the container exposes.

 **10. containerPort**: Indicates the port number that the container listens on. Here, it’s set to 80, which is the default port for HTTP traffic.

 ## Deleting a pod by name
 By deleting a pod, you’re instructing Kubernetes to terminate all the containers that are
part of that pod. Kubernetes sends a SIGTERM signal to the process and waits a certain
number of seconds (30 by default) for it to shut down gracefully. If it doesn’t shut down
in time, the process is then killed through SIGKILL. To make sure your processes are
always shut down gracefully, they need to handle the SIGTERM signal properly. 

## Requesting resources for a pod’s containers
When creating a pod, you can specify the amount of CPU and memory that a container needs (these are called requests) and a hard limit on what it may consume
(known as limits). They’re specified for each container individually, not for the pod as
a whole. The pod’s resource requests and limits are the sum of the requests and limits of all its containers. 

![image](https://github.com/user-attachments/assets/0155058c-b1ba-4ca4-b2ab-42315daf1b9f)

## UNDERSTANDING HOW THE SCHEDULER USES PODS’ REQUESTS WHEN SELECTING THE BEST NODE FOR A POD
Scheduler first filters the list of nodes to
exclude those that the pod can’t fit on and then prioritizes the remaining nodes per the
configured prioritization functions. Among others, two prioritization functions rank
nodes based on the amount of resources requested: LeastRequestedPriority and
MostRequestedPriority. The first one prefers nodes with fewer requested resources
(with a greater amount of unallocated resources), whereas the second one is the exact
opposite—it prefers nodes that have the most requested resources (a smaller amount of
unallocated CPU and memory). But, as we’ve discussed, they both consider the amount
of requested resources, not the amount of resources actually consumed.
 The Scheduler is configured to use only one of those functions. You may wonder
why anyone would want to use the MostRequestedPriority function. After all, if you
have a set of nodes, you usually want to spread CPU load evenly across them. However,
that’s not the case when running on cloud infrastructure, where you can add and
remove nodes whenever necessary. By configuring the Scheduler to use the MostRequestedPriority function, you guarantee that Kubernetes will use the smallest possible number of nodes while still providing each pod with the amount of CPU/memory
it requests. By keeping pods tightly packed, certain nodes are left vacant and can be
removed. Because you’re paying for individual nodes, this saves you money.

## Limit For The Amount Of Resources A Container Can Use
CPU is a compressible resource, which means the amount used by a container can
be throttled without affecting the process running in the container in an adverse way.
Memory is obviously different—it’s incompressible. Once a process is given a chunk of
memory, that memory can’t be taken away from it until it’s released by the process
itself. That’s why you need to limit the maximum amount of memory a container can
be given. 
 Without limiting memory, a container (or a pod) running on a worker node may
eat up all the available memory and affect all other pods on the node and any new
pods scheduled to the node (remember that new pods are scheduled to the node
based on the memory requests and not actual memory usage). A single malfunctioning or malicious pod can practically make the whole node unusable.

![image](https://github.com/user-attachments/assets/964409e0-e54a-4336-9423-ac9741cebe85)

