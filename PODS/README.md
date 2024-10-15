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
