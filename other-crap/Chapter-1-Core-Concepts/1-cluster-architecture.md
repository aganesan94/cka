# Cluster Architecture

References: https://kubernetes.io/docs/concepts/architecture/

* Deploying K8s gets a cluster.
* A cluster consists of a set of nodes.
* A cluster consists of ATLEAST ONE NODE
* Nodes are of 2 types:
  * Master node
  * Worker Node

## Master node

* Consists of the control plane.
* It manages the workloads on the worker node.


### Control Plane Components

* Make global decisions about the cluster (example, schedule, detecting and responding to events)
* Can be run on any machine in the cluster. But typically all components are started on the same machine.

#### kube-API Server

 - Front end for the kubernetes control plane.
 - Implementation is the kube-apiserver
 - kube-api server scales horizontally b deploying more instances.

#### etcd

- Key Value store to bank the K8 related data.
- Administrator should decide on back up plan in case we plan to upgrade the clusters and use etcd
 
#### kube-scheduler

- Control plane component that watches for newly created pods and assigns them to nodes.
- This involves taking into account hardware/software/policy constraints, affinity and anti affinity specifications.

#### kube-proxy

- Maintains network rules on nodes.
- Network rules allow network communication to pods from network sessions, inside or outside of cluster.

#### Container runtime

- Responsible for running containers.

## Worker node

* Runs pods
* Virtual or Physical machine depending on the cluster
* Contains the services required to run pods.

Node contains the following components

* Kube Proxy
  * Maintains network rules on each node.
  
* Kubelet 
  * Responsible for running pods inside a container
  
* Container runtime
  * Ensuring running a container
