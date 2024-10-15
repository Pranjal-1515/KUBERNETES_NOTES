# REPLICATION CONTROLLER
A ReplicationController ensures that a specified number of pod replicas are running at any one time. 
In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.<br>
A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, 
the ReplicationController supervises multiple pods across multiple nodes.

## Introducing liveness probes
Kubernetes can check if a container is still alive through liveness probes. You can specify a liveness probe for each container in the pod’s specification. 
Kubernetes will periodically execute the probe and restart the container if the probe fails.<br>
Kubernetes can probe a container using one of the three mechanisms:

* An HTTP GET probe performs an HTTP GET request on the container’s IP address, a port and path you specify. If the probe receives a response, and the
  response code doesn’t represent an error (in other words, if the HTTP response
  code is 2xx or 3xx), the probe is considered successful. If the server returns an
  error response code or if it doesn’t respond at all, the probe is considered a failure and the container will be restarted as a result.

* A TCP Socket probe tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful.
  Otherwise, the container is restarted.

* An Exec probe executes an arbitrary command inside the container and checks the command’s exit status code. If the status code is 0, the probe is successful.
  All other codes are considered failures.
