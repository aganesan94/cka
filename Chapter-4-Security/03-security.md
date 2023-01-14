# TLS

Certificates have 2 tasks: 
* Encryption of data between client and server.
* Server is who it says it is.

![Alt text](images/security.png "Security")

### Certificate used by different components

```shell
cat /etc/kubernetes/manifests/etcd.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

### To Find more information about the certificate

```shell
openssl x509 -in <path-to-file>.crt -text -noout
```


### Working with certificates
```shell
# Approving or denying a certificate
k certificate approve/deny <csr-name>

# Get a CSR
k get csr

# Inspect a CSR
k get csr <csr-name> -o yaml

# Delete a CSR
k delete csr <csr-name>
```

### Working with kubeconfig

```shell

# Setting the path of kubeconfig
export KUBECONFIG=path1:path2:path3

# Viewing the kube config
kubectl config view
kubectl config view --kubeconfig=<custom-config>

# Changing the context 
kubectl config use-context <context-name>
kubectl config use-context --kubeconfig=<custom-config> <context-name>

# Get the current context
kubectl config get-contexts
kubectl config get-contexts --kubeconfig=<custom-config> 

# Modifying the context
kubectl config set-context \
<CONTEXT_NAME> \
--namespace=<NAMESPACE_NAME> \
--cluster=<CLUSTER_NAME> \
--user=<USER_NAME>

kubectl config set-context --kubeconfig=<custom-config>\
<CONTEXT_NAME> \
--namespace=<NAMESPACE_NAME> \
--cluster=<CLUSTER_NAME> \
--user=<USER_NAME>

# Use the context
kubectl config use-context

```