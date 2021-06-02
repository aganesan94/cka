# KUBE API Server

* The following command fetches the information from the api server

```
kubectl get nodes
```

* The request is first authenticated with the API server, validated and data is retrieved.

Interaction

```
Client <---> API Server <---> etcd Cluster
```

* When a new pod is created

Interaction
```

Client <---> API Server <---> etcd Cluster
Scheduler <---> Identifies the node < - > API Server
API Server <----> KUbelet (instructed to create the pod in the node, and sends the info back to the API cluster)
API Server <---> updates the etcd

```


When the API server is deployed via the kubeadmin tool, the api-server deploys the api server as a pod in the kube-system namespace.