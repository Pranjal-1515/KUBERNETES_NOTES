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

##  Creating a ReplicationController
![image](https://github.com/user-attachments/assets/0862d7a3-8676-459f-a63a-552af9d5daaf)

* Although a pod isn’t tied to a ReplicationController, the pod does reference it in the metadata.ownerReferences field, which you can use to easily
  find which ReplicationController a pod belongs to.

##  Horizontally scaling pods
You’ve seen how ReplicationControllers make sure a specific number of pod instances
is always running. Because it’s incredibly simple to change the desired number of replicas, this also means scaling pods horizontally is trivial. <br>
We can use the below command to scale replicas in rc:

```bash
   kubectl scale rc <rc-name> --replicas=10
```

* When deleting a ReplicationController with kubectl delete, you can keep its
pods running by passing the --cascade=false option to the command. <br>
```bash
   kubectl delete rc <rc-name> --cascade=false
```




  
