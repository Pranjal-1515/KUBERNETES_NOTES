# Readiness Probes
The readiness probe is invoked periodically and determines whether the specific pod should receive client requests or not. When a container’s readiness probe returns
success, it’s signaling that the container is ready to accept requests. This notion of being ready is obviously something that’s specific to each container.
Kubernetes can merely check if the app running in the container responds to a simple GET / request or it can hit a specific URL path, which causes the app to perform a
whole list of checks to determine if it’s ready. Such a detailed readiness probe, which takes the app’s specifics into account, is the app developer’s responsibility.

## Types Of Readiness Probes

*  **An Exec probe,** where a process is executed. The container’s status is determined by the process’ exit status code.

*  **An HTTP GET probe,** which sends an HTTP GET request to the container and the HTTP status code of the response determines whether the container is ready or not.

*  **A TCP Socket probe,** which opens a TCP connection to a specified port of the container. If the connection is established, the container is considered ready.

## Understanding The Operation Of Readiness Probes
When a container is started, Kubernetes can be configured to wait for a configurable amount of time to pass before performing the first readiness check. After that, it
invokes the probe periodically and acts based on the result of the readiness probe. If a pod reports that it’s not ready, it’s removed from the service. If the pod then becomes
ready again, it’s re-added. <br><br> Unlike liveness probes, if a container fails the readiness check, it won’t be killed or
restarted. This is an important distinction between liveness and readiness probes. Liveness probes keep pods healthy by killing off unhealthy containers and replacing
them with new, healthy ones, whereas readiness probes make sure that only pods that are ready to serve requests receive them.



  
