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
| API Server  | 6443      |
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

#Enable kubelet
sudo systemctl enable --now kubelet

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
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml   
```