# Volumes
Kubernetes volumes are a component of a pod and are thus defined in the pod’s specification—much like containers. They aren’t a standalone Kubernetes object and cannot be created or deleted on their own. A volume is available to all containers in the
pod, but it must be mounted in each container that needs to access it. In each container, you can mount the volume in any location of its filesystem

##  Using an emptyDir Volume
emptyDir is a type of volume in Kubernetes that provides a temporary storage space for a pod. It is created when a pod is assigned to a node and exists as long as that pod is running. <br>
When you mount an emptyDir volume to a container at a specific path like /var, the data stored in that emptyDir will be available at that same path inside the container. However, on the node, you can find the underlying data at the path mentioned
```bash
  /var/lib/kubelet/pods/<pod-uid>/volumes/kubernetes.io~empty-dir/<volume-name>
```
## hostPath Volume
A hostPath volume points to a specific file or directory on the node’s filesystem. Pods running on the same node and using the same path in their hostPath volume see the same files. <br>
hostPath volumes are the first type of persistent storage we’re introducing, because emptyDir volumes’ contents get deleted when a pod is torn
down, whereas a hostPath volume’s contents don’t. If a pod is deleted and the next
pod uses a hostPath volume pointing to the same path on the host, the new pod will
see whatever was left behind by the previous pod, but only if it’s scheduled to the same
node as the first pod.

## PersistentVolumes and PersistentVolumeClaims

**PV (Persistent Volume):** A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PVs are independent of individual Pods and exist beyond their lifecycle.

**PVC (Persistent Volume Claim):** A request for storage by a user. It specifies the desired size and access modes. PVCs allow users to claim storage resources without needing to understand the underlying infrastructure.

## Understanding the benefits of using PersistentVolumes and claims
Consider how using this indirect method of obtaining storage from the infrastructure
is much simpler for the application developer (or cluster user). Yes, it does require
the additional steps of creating the PersistentVolume and the PersistentVolumeClaim,
but the developer doesn’t have to know anything about the actual storage technology
used underneath.  <br>
 Additionally, the same pod and claim manifests can now be used on many different
Kubernetes clusters, because they don’t refer to anything infrastructure-specific. The
claim states, “I need x amount of storage and I need to be able to read and write to it
by a single client at once,” and then the pod references the claim by name in one of
its volumes.

## StorageClass
A StorageClass in Kubernetes is a way to define different types of storage that can be dynamically provisioned for persistent volumes. It provides a way to describe the characteristics of the storage, such as performance tiers, volume type, or specific features like encryption or replication.

