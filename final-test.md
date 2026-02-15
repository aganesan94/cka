# Final walk thru

## Imperative commands

```shell
# Create the pod
kubectl run <pod-name> --image=<image> --dry-run=client -o yaml

# Create the deployment
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

# Create service
kubectl create service nodeport <service-name> .....

# API resources
kubectl api-resources

# Explaining a resource
kubectl explain pods --recursive

# Exposing a pod
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP

# Creating a pod and service together 
kubectl run httpd --image=httpd:alpine --port=80 --expose 


# Selecting pods with label
k get po -A --selector env=dev

# Using multiple labels
k get po --selector env=prod --selector bu=finance --selector tier=frontend

# label nodes
kubectl label nodes <your-node-name> disktype=ssd

# When adding commands, add to the end
k run static-busybox --image=busybox --dry-run=client -o yaml -- sh -c sleep 1000
```


## Commands in a pod

### Basic command in a pod

```yaml
---
apiVersion: apps/v1
kind: deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```

### If pod is in pending state the following items to check

* Node
* Taint and toleration
* Availability of the scheduler

### Creating a daemonset

1. Create a deployment using an imperative command, store in a file
2. Remove strategy, status and change kind to DaemonSet (Note D and S are in caps)

### Static pods
1. Pod Name has node name appended to it.
2. 2 pods which are usually not static:
* kube-proxy
* coredns
3. Static pods are usually created in /etc/kubernetes/manifests
4. Static pods can also be created on the worker node , there is a staticPodPath attribute in the /var/lib/kubelet/config.yaml file which specifies the directory of the static path.

### Priority Classes
1. A pod can define a priority class using priorityClassName
2. Pre-emption policy is of 2 types

* PreemptyLowerPriority - Remove existing pods with lower priority and replace pods with greater priority
* Never - do not do anything to the pods