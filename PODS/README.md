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
because the image wasn’t available locally. After downloading the image, Docker created and ran the container
