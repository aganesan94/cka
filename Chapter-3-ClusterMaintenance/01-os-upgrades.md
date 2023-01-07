# Upgrading a cluster

* Drain a node - Terminate the pods and recreate the same pods on another node. This also mark the node as unschedulable
* Cordon a node - Mark the node as unschedulable, no pods are scheduled on this node.
* Uncordon a node - Mark the node as scheduleable, new pods are created on this node.

Consider the above scenario where there is one Master and three workers. One of the worker nodes goes down

	- The blue pod in this case exists on another node so the blue pods are still accessible.
	- The green pod on the other hand existed only on the node which went down hence the users accessing the green pod are impacted.
	- What are the options at the k8s level?
		○ If the node came back online, the control process starts and the pods come back online
		○ If the node is down for more than 5 minutes (pod eviction timeout determines this and is set in the controller manager), the pods are terminated from that node.
			§ If the pod is created from a replica set, the pods are created on other nodes

Note: Creating a replicaset is ideal.

```shell
# Drains all the pods on one node and can be used to recreate them on another node
# Ensure the node is cordoned and marked as unscheduled
# The drain node does not work if the node has pods which are not created as a part of the replicaset
kubectl  drain node --ignore-daemonsets
```

```shell
# If you need to mark the node as schedulable for new pods
kubectl uncordon node
```

```shell
# Marking a schedule unschedulable
kubectl cordon node
```