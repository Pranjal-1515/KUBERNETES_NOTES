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

## Session Affinity On The Service     
If you execute the same command a few more times, you should hit a different pod
with every invocation, because the service proxy normally forwards each connection
to a randomly selected backing pod, even if the connections are coming from the
same client. <br>
 If, on the other hand, you want all requests made by a certain client to be redirected to the same pod every time, you can set the service’s sessionAffinity property
to ClientIP (instead of None, which is the default), This makes the service proxy redirect all requests originating from the same client IP
to the same pod. As an exercise, you can create an additional service with session affinity set to ClientIP and try sending requests to it. <br>
 Kubernetes supports only two types of service session affinity: None and ClientIP. You may be surprised it doesn’t have a cookie-based session affinity option, but you
need to understand that Kubernetes services don’t operate at the HTTP level. Services deal with TCP and UDP packets and don’t care about the payload they carry. Because
cookies are a construct of the HTTP protocol, services don’t know about them, which explains why session affinity cannot be based on cookies.

## Exposing Multiple Ports In The Same Service
Services can also support multiple ports. For example, if your pods listened on two ports—let’s say 8080 for HTTP and 8443 for
HTTPS—you could use a single service to forward both port 80 and 443 to the pod’s ports 8080 and 8443. You don’t need to create two different services in such cases. Using
a single, multi-port service exposes all the service’s ports through a single cluster IP

## Endpoints
In Kubernetes, an Endpoint is an object that represents a set of network endpoints (IP addresses and ports) for a service. When you create a Service in Kubernetes, it abstracts a group of Pods that provide the same functionality. The Endpoint object maintains the list of IP addresses of the Pods that match the Service's selector. <br>
Although the pod selector is defined in the service spec, it’s not used directly when redirecting incoming connections. Instead, the selector is used to build a list of IPs
and ports, which is then stored in the Endpoints resource. When a client connects to a service, the service proxy selects one of those IP and port pairs and redirects the
incoming connection to the server listening at that location.

## Exposing Services To External Clients

* **Setting the service type to NodePort:** For a NodePort service, each cluster node
opens a port on the node itself (hence the name) and redirects traffic received
on that port to the underlying service. The service isn’t accessible only at the
internal cluster IP and port, but also through a dedicated port on all nodes. 

* **Setting the service type to LoadBalancer, an extension of the NodePort type:** This
makes the service accessible through a dedicated load balancer, provisioned
from the cloud infrastructure Kubernetes is running on. The load balancer redirects traffic to the node port across all the nodes. Clients connect to the service
through the load balancer’s IP.
 
* **Creating an Ingress resource, a radically different mechanism for exposing multiple services through a single IP address:** It operates at the HTTP level (network layer 7)
and can thus offer more features than layer 4 services can.

## Using a NodePort service
The first method of exposing a set of pods to external clients is by creating a service
and setting its type to NodePort. By creating a NodePort service, you make Kubernetes
reserve a port on all its nodes (the same port number is used across all of them) and
forward incoming connections to the pods that are part of the service. 
This is similar to a regular service (their actual type is ClusterIP), but a NodePort
service can be accessed not only through the service’s internal cluster IP, but also
through any node’s IP and the reserved node port. 
This will make more sense when you try interacting with a NodePort service.

## Examining Your Nodeport Service 
Look at the EXTERNAL-IP column. It shows <nodes>, indicating the service is accessible
through the IP address of any cluster node. The PORT(S) column shows both the
internal port of the cluster IP (80) and the node port (30123). The service is accessible at the following addresses:

* 10.111.254.223:80
* <1st node’s IP>:30123
* <2nd node’s IP>:30123, and so on.
  
Your service exposed on port 30123 of both of your cluster nodes.
An incoming connection to one of those ports will be redirected to a randomly selected pod, which may or may not be the one running on the
node the connection is being made to.

## Exposing A Service Through An External Load Balancer
Kubernetes clusters running on cloud providers usually support the automatic provision of a load balancer from the cloud infrastructure. All you need to do is set the
service’s type to LoadBalancer instead of NodePort. The load balancer will have its own unique, publicly accessible IP address and will redirect all connections to your
service. <br> You can thus access your service through the load balancer’s IP address. If Kubernetes is running in an environment that doesn’t support LoadBalancer
services, the load balancer will not be provisioned, but the service will still behave like
a NodePort service. That’s because a LoadBalancer service is an extension of a NodePort service

![image](https://github.com/user-attachments/assets/f01ba64a-b4da-47d9-8c9e-340ad11ae9b8)


## Session Affinity And Web Browsers
Because your service is now exposed externally, you may try accessing it with your web browser. You’ll see something that may strike you as odd—the browser will hit
the exact same pod every time. Did the service’s session affinity change in the meantime? With kubectl describe, you can double-check that the service’s session
affinity is still set to None, so why don’t different browser requests hit different pods, as is the case when using curl? <br>
Let me explain what’s happening. The browser is using keep-alive connections and sends all its requests through a single connection, whereas curl opens a new
connection every time. Services work at the connection level, so when a connection to a service is first opened, a random pod is selected and then all network packets belonging
to that connection are all sent to that single pod. Even if session affinity is set to None, users will always hit the same pod (until the connection is closed).

## Exposing Services Externally Through An Ingress Resource
One important reason is that each LoadBalancer service requires its own load balancer with its own public IP address, whereas an Ingress only requires one, even when
providing access to dozens of services. When a client sends an HTTP request to the Ingress, the host and path in the request determine which service the request is forwarded to. <br>

**Note:** Ingresses operate at the application layer of the network stack (HTTP) and can provide features such as cookie-based session affinity and the like, which services can’t.








 
