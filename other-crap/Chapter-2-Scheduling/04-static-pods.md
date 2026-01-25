# Static Pods

Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. 
Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it fails).
Static Pods are always bound to one Kubelet on a specific node.
The kubelet automatically tries to create a mirror Pod on the Kubernetes API server for each static Pod. 
This means that the Pods running on a node are visible on the API server, but cannot be controlled from there. The Pod names will suffixed with the node hostname with a leading hyphen

There are 2 ways to create static pods

* Filesystem-hosted static Pod manifest

  * The pod definition is placed *ON a node* and */etc/kubelet.d*
  * Edit the following file 

  ```
  vim /etc/kubernetes/kubelet
  
  #Add
  KUBELET_ARGS="--cluster-dns=10.254.0.10 --cluster-domain=kube.local --pod-manifest-path=/etc/kubelet.d/"
  ```

```
# Use the staticPodPath: <the directory> field in the kubelet 
# configuration file, which periodically scans the directory and 
# creates/deletes static Pods as YAML/# JSON files appear/disappear there

staticPodPath: /etc/kubelet.d/

```


* Web-hosted static pod manifest

  ```
  vim /etc/kubernetes/kubelet
  
  #Add
  KUBELET_ARGS="--cluster-dns=10.254.0.10 --cluster-domain=kube.local --manifest-url=<manifest-url>"
  ```


### Viewing static pods

The following commmand helps us to view the docker pods.

```
docker ps
```

You can see the mirror Pod on the API server

```
kubectl get pods
```

### Use cases

* Running the control plane on each node if required is a good use case.
  
### Identifying static pods


* The pods which end with "-controlplane"
* kube-proxy is not deployed as a static pod.
* kube-controller-manager, etcd-controlplane, kube-apiserver can be deployed as static pods.


### Finding the static pod definition path

```
# Find the kubelet process

ps aux | grep kubelet

# Locate the --config parameter

root      5024  0.0  0.1 4076824 107772 ?      Ssl  00:19   0:56 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2

# Locate staticPodPath: in 
cat /var/lib/kubelet/config.yaml 

# Locate the manifests
root@controlplane:~# ls -ltr /etc/kubernetes/manifests
total 16
-rw------- 1 root root 3314 Jun  3 00:17 kube-controller-manager.yaml
-rw------- 1 root root 3812 Jun  3 00:17 kube-apiserver.yaml
-rw------- 1 root root 1384 Jun  3 00:17 kube-scheduler.yaml
-rw------- 1 root root 2189 Jun  3 00:17 etcd.yaml


# Adding a pod to the above path automatically starts a start pod, removing the file automatically deletes the pod
k run static-busybox --image=busybox:1.28.4 --dry-run=client -o yaml --command -- /bin/sh -c 'sleep 1000' > /etc/kubernetes/manifests/busybox-pod.yaml

```