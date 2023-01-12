# TLS

Certificates have 2 tasks: 
* Encryption of data between client and server.
* Server is who it says it is.

![Alt text](images/tls.png "a title")

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