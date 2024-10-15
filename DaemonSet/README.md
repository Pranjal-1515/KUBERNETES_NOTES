# DaemaonSet
A DaemonSet is a Kubernetes resource that ensures a copy of a specific pod runs on all (or some) nodes in a cluster. 
It is useful for deploying system-level services, such as log collectors, monitoring agents, or network proxies, that need to run on every node or a subset of nodes.

## Key Features:

 * **Automatic Pod Management:** When a new node joins the cluster, a DaemonSet automatically schedules the defined pod on that node.
 * **Node Selection:** You can control which nodes should run the DaemonSet using node selectors, taints, and tolerations.
 * **Updates**: DaemonSets can be updated, and the pods will be gradually rolled out across the nodes.

## Creating DaemonSet from YAML
![image](https://github.com/user-attachments/assets/cf6d9467-f6de-4d78-84ac-03c2b52522dc)
