# Cluster upgrade

During the cluster upgrade process the following requirements are to be satisfied

* kube-apiserver is the most important part of the k8 cluster
* No other component should be greater than the version of the kube-apiserver.
* The kube-controller-manager and kube-scheduler should be atleast one version lower than or equal to the kube-apiserver
* The kubelet and kube proxy should be atleast two versions lower or equal to the kube-api server
* Kubernetes would support only the current version and the last 2 versions from the current version at any given time.

## Example of different versions

kube-api-server => v1.10

kube-controller-manager => v1.9 or v.10
kube-scheduler => v1.9 or v.10

kubelet => v1.8, v1.9, v1.10
kube-proxy => v1.8, v1.9, v1.10

  
## Upgrade Process

* K8 Upgrades should be upgraded 1 minor version at a time. So for example if the current version is 1.13 and we are currently under 1.10, upgrade
  from 1.10 to 1.11 to 1.12 to 1.13
* Upgrade master nodes first and then upgrade worker nodes.

### Strategies

* Upgrade kubeadm tool first
* Upgrade all at once.
* Upgrade one node at a time.

```shell
# Drain the node first to ensure all pods are rescheduled on other nodes
# and no new pods can be scheduled on the current node
kubectl drain <node>

# Checking if k8s cluster was installed via kubeadm
kubeadm token list

# Upgrade kubeadm
apt-get upgrade -y kubeadm=<version-to-upgrade>

# Upgrading the kubelet
apt-get upgrade -y kubelet=<version-to-upgrade>

# Upgrade the new kubelet version on the node
kubeadm upgrade node config --kubelet-version <version-to-upgrade>

# Restart kubelet after upgrade
systemctl restart kubelet

# Specifies the plan to upgrade
kubectl upgrade plan

# Upgrade
kubectl upgrade apply <version-to-upgrade>

# Required to restart all services
systemctl daemon-reload
systemctl restart kubelet

# Uncordon the node to ensure new pods can be scheduled
kubectl uncordon <node>
```