# kubeadm tool

### Master
---
Master consists of the following components:

* kube-apiserver
* etcd
* node-controller
* replica-controller

### Master
---
Worker consists of the following components:

* kubelet
* Container runtime

---

## Steps to Install

1. Provision multiple VM's to configure a cluster.
2. Designate one node as master and others as workers nodes
3. Install Docker on all the nodes.
4. Install kubeadm tool 
5. Initialize master by installing all the components
6. Join worker nodes on master
7. A special network installation is required between master to the worker node.