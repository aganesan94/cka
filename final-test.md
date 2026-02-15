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