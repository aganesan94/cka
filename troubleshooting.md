# Troubleshooting - 30%

## JSON path queries

* kubectl is a cli tool which authenticates and authorizes with the API server and to query to get the required information.
* The information is provided by the API server to kubectl in a json format.
* kubectl filters the required information and presents it.
* However, this mechanism can be enhanced to provide more information in a printable format

### Reference 

* https://kubernetes.io/docs/reference/kubectl/jsonpath/
* https://kubernetes.io/docs/reference/kubectl/ - Check on custom columns
---

## General 

```shell
# Returns all resources
kubectl get all -A

# Get all resources in a given namespace
kubectl get all -n <namespace>
```

### Application

#### Checks
* Name of the pods and the services
* Ports of the ports and the services
* Selectors on the service and the pod
* Volume mounts
* Describe pods
* Check events
* Check logs
* On access denied - Check db params 
* On unable to connect - check service: name, selectors, ports
### Control Plane
* Check nodes
* Check pods in all namespaces and specifically the kube-system namespace
* Ensure to check if in the namespace the following pods are running
  * controller-manager
  * api-server
  * scheduler
  * etcd
  * coredns
  * networking
  * proxy
* If the pods are not running, check logs, describe pods and events
* If the above are not running as pods, they should be running as services
* Ensure to check if kubelet is running. Kubelet runs as a service
* If everything looks ok and a file is not found, check the volume mount
* Check if kubelet is running as a service

```shell
# Check kubelet running status
service kubelet status

# Logs of the kubelet service
journalctl -f -u kubelet

# All configuration files of kubelet are stored in
cat /var/lib/kubelet/config.yaml

# At times it may also be required to check ports in worker node
cat /etc/kubernetes/manifests

# Any changes to files in /var/lib/kubelete or /etc/kubernetes/ will require a restart
service kubelet restart
```

### Worker Node
* Check kubelet status as described in the control plane section
* Check certificates to make sure they are still valid
```shell
openssl x509 -in <location-to-cert> -text
```

### Networking

```shell
# Contains all the cni related files
ls -la /etc/cni/net.d/
```

* Check kube-proxy at all times to ensure it is running
* kube-proxy contains a configmap which describes the location to the config file
* The configmaps are volume mounted to daemonset. So the name of the config key in the config map will need to be set in the daemonset

# Reference
* https://kubernetes.io/docs/tasks/debug/debug-application/
* https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

### Reference
* https://kubernetes.io/docs/reference/kubectl/quick-reference/
