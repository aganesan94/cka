# Scheduling

## Manual Scheduling

* Used whenwe do not have a scheduler
* Pod can be scheduled manually via nodeName
* Whether a scheduler is available or not can be found using the

```shell
k get po -n kube-system
```

## Labels and selectors

* Used to group resources together.
* To fetch resources with labels

```shell
# Single label query
kubectl get pods --selector=key=value

# Multiple label query
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

* Refer to cheatsheet for adding/removeand overriding labels.
* https://kubernetes.io/docs/reference/kubectl/quick-reference/

## Auto Scheduling
### Taints and Tolerations

* Adding a nodename bypass the scheduler.
* Taint: A node which is tainted does not schedule a pod with the 
* Toleration:  A tag added to the pod which allows for pod to be placed in a node with a particular taint
* Adding a taint
* Removing a taint
* Taint has the following effects
  * NoExecute
  * NoSchedule
  * PreferNoSchedule

### Node Selectors

* Typically used when there is a user where the pods are to be scheduled on nodes requiring higher resources
* Works based on labels
* Automatically pod placement on node.

### Node affinity

* Pods are hosted only on particular nodes
* 2 types
  * requiredDuringSchedulingIgnoredDuringExecution: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
  * preferredDuringSchedulingIgnoredDuringExecution: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

### Resource Limits

* Placing pods based on the resource requirements
* OOMKilled,Crashloopbackoff -memory overload

```yaml
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Daemon Sets
* Always describe a daemon set to understand how many nodes have pods running on them.

### Static Pods

* Found in /etc/kubernetes/manifests directory
* The pods running as static pods have the node names suffixed.

### Priority Classes

* Default priority class is zero
* When pods of higher priority need to be scheduled, what happens is that the preemption policy determines if the pods of lower priority are evicted or if the current pods get added to queue

### Multiple schedulers

Refer https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/


### Admission Controllers

* Added as a configuration to the kube-apiserver.yaml file in /etc/kubernetes/manifests
* Adds an extra layer after authentication and authorization to validate something.
* 2 types
  * Validation AC
  * Mutation AC
* First mutate and validate
Refer https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
