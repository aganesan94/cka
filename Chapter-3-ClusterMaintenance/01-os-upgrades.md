# Upgrading a cluster

Drain a node - Terminate the pods and recreate the same pods on another node.
Cordon a node - Mark the node as unscheduleable, no pods are scheduled on this node.
Uncordon a node - Mark the node as scheduleable, new pods are created on this node.

## Emptying a node and making it unscheduleable

```
k cordon <node-name>
k drain <node-name> --ignore-daemonsets
 
```


* Trying to drain a node where a pod is not part of the replicaset
 will cause an error such as thiis in which case we need to use the --force flag wile draining. Such pods will be lost completely on drain

```
kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
error: unable to drain node "node01", aborting command...

There are pending nodes to be drained:
 node01
error: cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): default/hr-app
```

Hence use the --force flag
```
kubectl drain node01 --ignore-daemonsets --force
```


## Node shows up after being cordoned and drained

```
root@controlplane:~# k get nodes
NAME           STATUS                     ROLES                  AGE   VERSION
controlplane   Ready                      control-plane,master   18m   v1.20.0
node01         Ready,SchedulingDisabled   <none>                 16m   v1.20.0
```

## Node marked for scheduling again

> Note: 

* No pods will be scheduled on uncordon. Existing pods will continue to run on their nodes, however new pods will be scheduled

```
k uncordon <node-name>
```
