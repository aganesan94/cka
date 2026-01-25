# Multiple Schedulers

Reference: https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

* K8 ships with a default scheduler.
* Implementation of a custom scheduler is possible.
* Running multiple schedulers alongside the default scheduler is achievable.
* Pod deployment definition schedules the pod using the default scheduler if none is specified.


* *schedulerName* is an important property

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: my-scheduler
  containers:
  - name: pod-with-second-annotation-container
    image: k8s.gcr.io/pause:2.0
```

* If multiple copies of a scheduler pod are running on multiple master nodes, one needs to be elected as a leader using the *--leader-elect=true* option. A value of true would mean multiple masters with one leader in which case 2 other properties need to be set

```
--lock-object-namespace=<lock-object-namespace>
--lock-object-name=<lock-object-name>
```

* The scheduler is schedule in the *kube-system* namespace.
* The following command provides all the information on which scheduler picked the pod and had it scheduled on a node

```
k get ev
```