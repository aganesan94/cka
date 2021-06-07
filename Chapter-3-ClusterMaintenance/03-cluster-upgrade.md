# Cluster upgrade

During the cluster upgrade process the following requirements are to be satisfied

* kube-apiserver is the most important part of the k8 cluster
* No other component should be greater than the version of the kube-apiserver.
* The kube-controller-manager and kube-scheduler should be atleast one version lower than or equal to the kube-apiserver
* The kubelet and kube proxy should be atleast two versions lower or equal to the kube-api server

## Example:

kube-api-server => v1.10

kube-controller-manager => v1.9 or v.10
kube-scheduler => v1.9 or v.10

kubelet => v1.8, v1.9, v1.10
kube-proxy => v1.8, v1.9, v1.10

> Note:

* kubectl -> v1.9 or v1.10 or v1.11
* Kubernetes would support only the current version and the last 2 versions from the current version at any given time.
* K8 Upgrades should be upgraded 1 minor version at a time. So for example if the current version is 1.13 and we are currently under 1.10, upgrade
  from 1.10 to 1.11 to 1.12 to 1.13

  # Procedure to upgrade the cluster

  * https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/