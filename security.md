# Security

## Service Account

* A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster. Application Pods, system components, and entities inside and outside the cluster can use a specific ServiceAccount's credentials to identify as that ServiceAccount
* Associating it to a pod is done using serviceAccountName
* automountServiceAccountToken* is assigned either at the pod level or the service level
* kubelet, when a pod is created 
  * Automatically creates a token
  * Associates it to the pod by mounting it to a volume
  * Automatically rotates the token
  * Deletes the token when pod is deleted
* Creating a token manually has an expiry date of an hour which can be changed via the --duration flag 
* Every namespace has a default service account
* A service account is automatically attached to the pod on creation.
* Service accounts are usually stored in /var/run/secrets
* Service accounts are like user account which can be associated to a role via a rolebinding
* When accessing a cluster via the API

```shell
curl https://<controlplane-ip>:6443/api -insecure --header "Authorization: Bearer <token>"
```

## Image Security

