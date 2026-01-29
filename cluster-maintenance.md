# Cluster Architecture Installation and Configuration

## 1. Kubeadmin Setting up a multinode cluster

### Pre-requisites

#### 1. 3 VMs - 1 control plane and 2 worker nodes
#### 2. The vms should have internet connectivity
#### 3. Ensure uniqueness

```shell
# To run commands to check ip configuraitons
sudo apt install -y net-tools

# Ensure unique ip addresses
ifconfig -a

# Have unique product ids
sudo cat /sys/class/dmi/id/product_uuid
```

#### 4. Ensure required ports are open

*controlplane*

| Service     | Port      |
|-------------|-----------|
| Controller  | 10257     |
ff| API Server  | 6443      |
| Scheduler   | 10259     |
| ETCD server | 2379-2380 |
| Kubelet     | 10250     |

*worker nodes*

| Service   | Port  |
|-----------|-------|
| Kubelet   | 10250 |
| kubeproxy | 10256 |

#### 5. Turn swap off

```shell
# Turn off swap temporarily
sudo swapoff -a

# Permanently switch off fstab
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Just in case ensure swap is turned off during reoobt
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```

#### 6. Manually turn on IPV4 packet forwarding

```shell
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Verify if set to 1
sysctl net.ipv4.ip_forward
```

#### 7. Install containerd

```shell
sudo apt install -y containerd
sudo systemctl daemon-reload
sudo systemctl enable containerd --now
sudo systemctl start containerd.service
```

#### 8. Generate config.toml

```shell
# Create containerd directory
sudo mkdir -p /etc/containerd/

# Generate the default containerd configuration
sudo containerd config default | sudo tee /etc/containerd/config.toml

# turn systemdcgroup to true
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd to apply changes
sudo systemctl restart containerd
```

#### 9. Install crictl

```shell
export CRICTL_VERSION=v1.34.0
export CRICTL_ARCH="amd64"

# Install crictl (amd64/arm64 based on system)
curl -LO "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-${CRICTL_ARCH}.tar.gz"
sudo tar zxvf "crictl-${CRICTL_VERSION}-linux-${CRICTL_ARCH}.tar.gz" -C /usr/local/bin
rm -f "crictl-${CRICTL_VERSION}-linux-${CRICTL_ARCH}.tar.gz"

# Check version
crictl --version
```

#### 10. Configure crictl to use containerd

```shell
# Configure crictl to use containerd
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

#### 11. Install keys and other associated packages

```shell
sudo apt-get update -y

# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg jq -y 
```
####  12. Install kubeadm, kubectl, kubelet 

```shell
export KUBERNETES_VERSION="v1.34"
export KUBERNETES_INSTALL_VERSION="1.34.0-1.1"

# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet="$KUBERNETES_INSTALL_VERSION" kubectl="$KUBERNETES_INSTALL_VERSION" kubeadm="$KUBERNETES_INSTALL_VERSION"

# Prevent automatic updates for kubelet, kubeadm, and kubectl
sudo apt-mark hold kubelet kubeadm kubectl

echo "KUBEADM Version: $(kubeadm version)"
echo "kubectl Version: $(kubectl version)"
echo "kubelet Version: $(kubelet --version)"

# Check what version are available
apt-cache madison kubelet
apt-cache madison kubeadm
apt-cache madison kubectl

```
####  13. Setting up the master node

```shell

# Pick a CIDR which is not the cidr of any of the nodes, meaning the node ip should exist within this range
kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/containerd/containerd.sock

```

[Important: Post Installation kubeadm](additional-resources/kubeadm-logs.md)


#### 14. Setting up calico

* This is the CNI

```shell
kubectl apply -f  https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/calico.yaml

# Ensure cni is installed correctly, notice calico files
ls -la /etc/cni/net.d/
total 16
drwx------ 2 root root 4096 Jan 26 05:57 .
drwxr-xr-x 3 root root 4096 Jan 26 03:33 ..
-rw-r--r-- 1 root root    0 Aug 28 17:00 .kubernetes-cni-keep
-rw------- 1 root root  584 Jan 26 05:57 10-calico.conflist
-rw------- 1 root root 2796 Jan 26 05:57 calico-kubeconfig

```

#### 15. Double check if nodes are ready in the control plane

##### Ensure node is ready
```shell
kubectl get nodes -o wide

NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   22m   v1.34.0   165.22.213.191   <none>        Ubuntu 24.04.3 LTS   6.8.0-71-generic   containerd://1.7.28
```

##### Ensure all pods are running

* Notice the pods running include
  * controller-manager
  * api-server
  * scheduler
  * etcd
  * coredns
  * networking (calico in this case)

```shell


kubectl get po -n kube-system

NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6dfb8bbfd4-sj4zj   1/1     Running   0          3m10s
calico-node-7hf5k                          1/1     Running   0          3m10s
coredns-66bc5c9577-mjx7z                   1/1     Running   0          23m
coredns-66bc5c9577-mvdjr                   1/1     Running   0          23m
etcd-control-plane-1                       1/1     Running   0          23m
kube-apiserver-control-plane-1             1/1     Running   0          23m
kube-controller-manager-control-plane-1    1/1     Running   0          23m
kube-proxy-v2rl4                           1/1     Running   0          23m
kube-scheduler-control-plane-1             1/1     Running   0          23m

```

#### 16. Setting up the worker node - 1

```shell
# Refer to join command in the post installation logs
kubeadm join 165.22.213.191:6443 --token at07se.0qu1ysne9uhv2tcs \
	--discovery-token-ca-cert-hash sha256:0ac795cf97cf7dd60048b9a3affae0a3446549192441db61be49dae90120b75d
```

[Important: Post Installation kubeadm on worker](additional-resources/kubeadm-logs.md)


##### Ensure node is ready
```shell
kubectl get nodes

NAME              STATUS   ROLES           AGE     VERSION
control-plane-1   Ready    control-plane   29m     v1.34.0
worker-node-1     Ready    <none>          3m52s   v1.34.0
```

#### 17. Install metrics server

```shell
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Edit and add the following line --kubelet-insecure-tls to
      k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
    
# Set it up
k create -f components.yaml    
```

#### 18. Install nginx

```shell
kubectl run nginx-pod --image=nginx
```

#### 19. Ensure metrics server is working

```shell
# kubectl top nodes
NAME              CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
control-plane-1   168m         8%       1143Mi          61%         
worker-node-1     149m         7%       917Mi           49% 
```

#### 20. Adding another worker node

* Repeat steps from step 1 to step 12
* The following need to be installed in the new worker-node-2

```shell
# This is should provide the script to use to join the k8s cluster
kubeadm token create --print-join-command

# Check and verify if the new node has joined the node
kubectl get nodes 
```

## 2. Upgrading the k8s cluster to the current release

### Order of upgrade

1. Control plane nodes
2. Worker nodes

### 1. Upgrade kubeadm on Control Plane

#### 1. Check current version of kubeadm
```shell
kubeadm version
```

* Note the version
```json
{
  "clientVersion": {
    "major": "1",
    "minor": "34",
    "gitVersion": "v1.34.0",
    "gitCommit": "f28b4c9efbca5c5c0af716d9f2d5702667ee8a45",
    "gitTreeState": "clean",
    "buildDate": "2025-08-27T10:15:59Z",
    "goVersion": "go1.24.6",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}
```

#### 2. Check the latest available version to upgrade

* The latest version is 1.34.3-1.1
```shell
apt-cache madison kubeadm

kubeadm | 1.34.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
```

#### 3. Upgrade the kubeadm version

```shell
export KUBERNETES_UPGRADE_VERSION=1.34.3-1.1

sudo apt-mark unhold kubeadm && \
sudo apt-get update -y && sudo apt-get install -y kubeadm="$KUBERNETES_UPGRADE_VERSION" && \
sudo apt-mark hold kubeadm

# decide on the upgrade version  
kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.34.3
[upgrade/versions] kubeadm version: v1.34.3
I0126 09:40:13.219638   56838 version.go:260] remote version is much newer: v1.35.0; falling back to: stable-1.34
[upgrade/versions] Target version: v1.34.3
[upgrade/versions] Latest version in the v1.34 series: v1.34.3

```
#### 4. Upgrade the kubeadm version

```shell
# Upgrade all the components.
kube upgrade apply v1.34.3 

# Check if all the versions have gotten upgraded
kubectl get nodes -o yaml | grep -i 'apiserver'
kubectl get nodes -o yaml | grep -i 'scheduler'
```

#### 5. Drain the controlplane

```shell
kubectl drain control-plane-1 --ignore-daemonsets
```

#### 6. Upgrade kubelet and kubectl

```shell
export KUBERNETES_UPGRADE_VERSION=1.34.3-1.1

# replace x in 1.34.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet="$KUBERNETES_UPGRADE_VERSION" kubectl="$KUBERNETES_UPGRADE_VERSION" && \
sudo apt-mark hold kubelet kubectl

# Check if kubectl and kubelet have been upgrade
kubectl version
kubelet --version

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

#### 7. Uncordon the control plane node
```shell
kubectl uncordon control-plane-1

# Get node status
# kubectl get nodes
NAME              STATUS   ROLES           AGE    VERSION
control-plane-1   Ready    control-plane   3h4m   v1.34.3
worker-node-1     Ready    <none>          3h2m   v1.34.0
worker-node-2     Ready    <none>          82m    v1.34.0
```

### 2. Upgrade kubeadm on Worker Node

* Only change in the worker node is we do not do upgrade apply
* We just run the upgrade command
* On running the below command no changes are reflected when you run *kubectl get nodes*

```shell

# Upgrade to latest kubeadm before running the below command

kubeadm upgrade node

[upgrade] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
[upgrade/preflight] Running pre-flight checks
[upgrade/preflight] Skipping prepull. Not a control plane node.
[upgrade/control-plane] Skipping phase. Not a control plane node.
[upgrade/kubeconfig] Skipping phase. Not a control plane node.
W0126 10:14:35.560818  116758 postupgrade.go:116] Using temporary directory /etc/kubernetes/tmp/kubeadm-kubelet-config3348452652 for kubelet config. To override it set the environment variable KUBEADM_UPGRADE_DRYRUN_DIR
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config3348452652/config.yaml
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade/kubelet-config] The kubelet configuration for this node was successfully upgraded!
[upgrade/addon] Skipping the addon/coredns phase. Not a control plane node.
[upgrade/addon] Skipping the addon/kube-proxy phase. Not a control plane node.
```

* Follow instructions of upgrade kubelet and kubectl as before
* Same instructions to be followedo on worker-node-2

### 3. Check if all services are up and running

```shell
kubectl get --raw='/readyz?verbose'

[+]ping ok
[+]log ok
[+]etcd ok
[+]etcd-readiness ok
[+]informer-sync ok
[+]poststarthook/start-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
[+]poststarthook/priority-and-fairness-config-consumer ok
[+]poststarthook/priority-and-fairness-filter ok
[+]poststarthook/storage-object-count-tracker-hook ok
[+]poststarthook/start-apiextensions-informers ok
[+]poststarthook/start-apiextensions-controllers ok
[+]poststarthook/crd-informer-synced ok
[+]poststarthook/start-system-namespaces-controller ok
[+]poststarthook/start-cluster-authentication-info-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-garbage-collector ok
[+]poststarthook/start-legacy-token-tracking-controller ok
[+]poststarthook/start-service-ip-repair-controllers ok
[+]poststarthook/rbac/bootstrap-roles ok
[+]poststarthook/scheduling/bootstrap-system-priority-classes ok
[+]poststarthook/priority-and-fairness-config-producer ok
[+]poststarthook/bootstrap-controller ok
[+]poststarthook/start-kubernetes-service-cidr-controller ok
[+]poststarthook/aggregator-reload-proxy-client-cert ok
[+]poststarthook/start-kube-aggregator-informers ok
[+]poststarthook/apiservice-status-local-available-controller ok
[+]poststarthook/apiservice-status-remote-available-controller ok
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-discovery-controller ok
[+]poststarthook/kube-apiserver-autoregistration ok
[+]autoregister-completion ok
[+]poststarthook/apiservice-openapi-controller ok
[+]poststarthook/apiservice-openapiv3-controller ok
[+]shutdown ok
readyz check passed

# kubectl get po -n kube-system 
NAME                                       READY   STATUS    RESTARTS      AGE
calico-kube-controllers-6dfb8bbfd4-chvcl   1/1     Running   0             5m29s
calico-node-bqkpm                          1/1     Running   0             3h20m
calico-node-vvzwh                          1/1     Running   0             3h20m
calico-node-z4hb9                          1/1     Running   0             101m
coredns-66bc5c9577-5zv7t                   1/1     Running   0             13m
coredns-66bc5c9577-qzjbz                   1/1     Running   0             5m29s
etcd-control-plane-1                       1/1     Running   0             41m
kube-apiserver-control-plane-1             1/1     Running   0             3h23m
kube-controller-manager-control-plane-1    1/1     Running   1 (42m ago)   3h23m
kube-proxy-4xx4m                           1/1     Running   0             101m
kube-proxy-nrwtr                           1/1     Running   0             3h23m
kube-proxy-sbwhs                           1/1     Running   0             3h21m
kube-scheduler-control-plane-1             1/1     Running   1 (42m ago)   3h23m
metrics-server-576c8c997c-8kd9r            1/1     Running   0             5m29s
```


## 3. Upgrading the k8s cluster - Next minor release

### 1. Modify current package repository

```shell
# On your system, this configuration file could have a different name
pager /etc/apt/sources.list.d/kubernetes.list

# Change it to the next minor release
cat /etc/apt/sources.list.d/kubernetes.list

# Comment this line out
# deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /

sudo apt update -y
```

### 2. Upgrade to the latest kubeadm

```shell
# Drain the node
kubectl drain control-plane-1 --ignore-daemonsets
```

```shell
export KUBERNETES_UPGRADE_VERSION="1.35.0-1.1"

sudo apt-mark unhold kubeadm && \
sudo apt-get update -y && sudo apt-get install -y kubeadm="$KUBERNETES_UPGRADE_VERSION" && \
sudo apt-mark hold kubeadm
```

```shell
# kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.34.3
[upgrade/versions] kubeadm version: v1.35.0
[upgrade/versions] Target version: v1.35.0
[upgrade/versions] Latest version in the v1.34 series: v1.34.3

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE              CURRENT   TARGET
kubelet     control-plane-1   v1.34.3   v1.35.0
kubelet     worker-node-1     v1.34.3   v1.35.0
kubelet     worker-node-2     v1.34.3   v1.35.0

Upgrade to the latest stable version:

COMPONENT                 NODE              CURRENT   TARGET
kube-apiserver            control-plane-1   v1.34.3   v1.35.0
kube-controller-manager   control-plane-1   v1.34.3   v1.35.0
kube-scheduler            control-plane-1   v1.34.3   v1.35.0
kube-proxy                                  1.34.3    v1.35.0
CoreDNS                                     v1.12.1   v1.13.1
etcd                      control-plane-1   3.6.5-0   3.6.6-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.35.0

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

```

### 3. Upgrade to the latest kubelet and kubectl

```shell
export KUBERNETES_UPGRADE_VERSION=1.35.0-1.1

# replace x in 1.34.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet="$KUBERNETES_UPGRADE_VERSION" kubectl="$KUBERNETES_UPGRADE_VERSION" && \
sudo apt-mark hold kubelet kubectl

# Check if kubectl and kubelet have been upgrade
kubectl version
kubelet --version

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### 4. Uncordon control-plane-1
```shell
kubectl uncordon control-plane-1
```

### 5. Upgrade the worker nodes as mentioned above.

## Flannel
[If flannel CNI is used](additional-resources/flannel.md)


# Cluster Maintenance

* If one of the worker nodes goes down and does not come back up in 5 minutes,
  the scheduler terminates the pods and moves them to another worker node.
* The pod eviction time out is 5 minutes by default

## Right step

1. Drain the node
2. update the OS
3. uncordon the node

* Note: The difference between cordon and drain is that cordon stops any pods
  from being scheduled on the node*

* After uncordoning the node only new pods will be scheduled
* When there is a pod in a node which is not a part of the deployment

# ETCD Snapshot

1. Stop etcd server
2. Take a snapshot
3. Check the status of the snapshot
4. stop kube-api-server
5. Restore the etcd cluster (Ensure the data is restored to a different data directory and the etcd.yaml updates to refer to the new directory)
6. Reload daemon
7. Restart etcd
8. Restart kube-api-server