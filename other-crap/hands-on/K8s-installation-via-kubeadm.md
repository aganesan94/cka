# What components are to be installed?

## On the Control Plane and Worker Node

1. kubelet - linux process
2. container runtime - linux process
3. kube-proxy - deployed as pod

## On the Control Plane only

1. Controller-Manager - deloyed as pod
2. API-Server - deployed as pod
3. Scheduler - deployed as pod
4. Etcd - deployed as pod


Note:

* The above pods are static pods via manifest files
* This is scheduled by the kubelet automatically
* Does not require a control plane
* Manifests are found in /etc/kubernetes/manifests

## Certificates

1. A CA (Root certificate) - self signed in generated
2. Sign all client and server certificates with the CA certificate.
3. All certificates are stores in /etc/kubernetes/certificates

## Steps

#### What needs to be installed first?

1. kubeadm
2. kubelet
3. kubectl

#### Pre-requisites

* Open required ports for control node and worker node
