# StatefulSets
A StatefulSet is a Kubernetes API object used to manage stateful applications.
Unlike a Deployment, which is suitable for stateless applications, a StatefulSet provides guarantees about the ordering and uniqueness of the pods it manages. 

## Comparing Statefulsets With Replicasets Or Replicationcontrollers
Pod replicas managed by a ReplicaSet or ReplicationController are much like cattle.
Because they’re mostly stateless, they can be replaced with a completely new pod
replica at any time. Stateful pods require a different approach. When a stateful pod
instance dies (or the node it’s running on fails), the pod instance needs to be resurrected on another node, but the new instance needs to get the same name, network
identity, and state as the one it’s replacing. This is what happens when the pods are
managed through a StatefulSet. <br>
 A StatefulSet makes sure pods are rescheduled in such a way that they retain their
identity and state. It also allows you to easily scale the number of pets up and down. A
StatefulSet, like a ReplicaSet, has a desired replica count field that determines how
many pets you want running at that time. Similar to ReplicaSets, pods are created from
a pod template specified as part of the StatefulSet (remember the cookie-cutter analogy?). But unlike pods created by ReplicaSets, pods created by the StatefulSet aren’t
exact replicas of each other. Each can have its own set of volumes—in other words,
storage (and thus persistent state)—which differentiates it from its peers. Pet pods
also have a predictable (and stable) identity instead of each new pod instance getting
a completely random one.

## Providing a stable network identity
Each pod created by a StatefulSet is assigned an ordinal index (zero-based), which
is then used to derive the pod’s name and hostname, and to attach stable storage to
the pod. The names of the pods are thus predictable, because each pod’s name is
derived from the StatefulSet’s name and the ordinal index of the instance. Rather
than the pods having random names, they’re nicely organized, as shown in the next
figure.

## Scaling A Statefulset
Scaling the StatefulSet creates a new pod instance with the next unused ordinal index.
If you scale up from two to three instances, the new instance will get index 2 (the existing instances obviously have indexes 0 and 1). 
 The nice thing about scaling down a StatefulSet is the fact that you always know
what pod will be removed. Again, this is also in contrast to scaling down a ReplicaSet,
where you have no idea what instance will be deleted, and you can’t even specify
which one you want removed first (but this feature may be introduced in the future).
Scaling down a StatefulSet always removes the instances with the highest ordinal index
first. This makes the effects of a scale-down predictable.

## INTRODUCING STATEFULSET’S AT-MOST-ONE SEMANTICS
Kubernetes must thus take great care to ensure two stateful pod instances are never
running with the same identity and are bound to the same PersistentVolumeClaim. A
StatefulSet must guarantee at-most-one semantics for stateful pod instances. 
 This means a StatefulSet must be absolutely certain that a pod is no longer running before it can create a replacement pod. This has a big effect on how node failures are handled. 
 We’ll demonstrate this later in the chapter. Before we can do that,
however, you need to create a StatefulSet and see how it behaves. You’ll also learn a
few more things about them along the way.

## SCALING A STATEFULSET
Scaling down a StatefulSet and scaling it back up after an extended time period
should be no different than deleting a pod and having the StatefulSet recreate it
immediately. Remember that scaling down a StatefulSet only deletes the pods, but
leaves the PersistentVolumeClaims untouched. I’ll let you try scaling down the StatefulSet yourself and confirm this behavior. 
The key thing to remember is that scaling down (and up) is performed gradually—similar to how individual pods are created when the StatefulSet is created initially. When scaling down by more than one instance, the pod with the highest ordinal
number is deleted first. Only after the pod terminates completely is the pod with the
second highest ordinal number deleted. 



