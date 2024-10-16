# Service
A Kubernetes Service is a resource you create to make a single, constant point of
entry to a group of pods providing the same service. Each service has an IP address
and port that never change while the service exists. Clients can open connections to
that IP and port, and those connections are then routed to one of the pods backing
that service. This way, clients of a service don’t need to know the location of individual pods providing the service, allowing those pods to be moved around the cluster
at any time.

## Service type

* **ClusterIP:** Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster.
This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.

* **NodePort:** Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available,
Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP.

*  **LoadBalancer:** Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component;
you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.

*  **ExternalName:** Maps the Service to the contents of the externalName field (for example, to the hostname api.foo.bar.example).
The mapping configures your cluster's DNS server to return a CNAME record with that external hostname value.No proxying of any kind is set up.

## Remotely Executing Commands In Running Containers
The kubectl exec command allows you to remotely run arbitrary commands inside
an existing container of a pod. This comes in handy when you want to examine the
contents, state, and/or environment of a container. List the pods with the kubectl
get pods command and choose one as your target for the exec command. You’ll also need to
obtain the cluster IP of your service (using kubectl get svc, for example). When running the following commands yourself, be sure to replace the pod name and the service IP with your own:
```bash
   $ kubectl exec pod-name -- curl -s http://10.111.249.153
     You’ve hit pod-name
```

* Why the double dash?

   * The double dash (--) in the command signals the end of command options for
     kubectl. Everything after the double dash is the command that should be executed
     inside the pod. Using the double dash isn’t necessary if the command has no
     arguments that start with a dash. But in your case, if you don’t use the double dash
     there, the -s option would be interpreted as an option for kubectl exec and would
     result in the misleading error
