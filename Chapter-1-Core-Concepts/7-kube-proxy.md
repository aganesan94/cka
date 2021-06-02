# Kube Proxy

* All the pods in all the nodes connect to a network cluster call the Pod Network.
* Runs on each node in the cluster.
* Scans for services in the cluster and creates appropriate policies for the pods to access the services.
* This is done thru IP table rules.