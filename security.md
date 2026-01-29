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

* Images can be downloaded from a repository <repositor>/accountname/packagename
* Create a secret of type docker-registry takes
  * server
  * username
  * password
  * email
* This is associated to the pod via imagePullSecrets

## Security Contexts

### Quick docker security overview
* Docker runs the container as a root user
* USER option is used to run the container with a userid
* Root user inside the container is not the same as the root user in the host.
* Capabilities can be added/removed via --cap-add --cap-drop 
* Privileged mode runs with full user permissions --privileged

* Pods are usually run as a root user with securityContext to empty by default.
* Need to know how to add a user and capabilities to run the pod
* securityContext can be provided
  * outside the containers section
  * within container section: overrides the general one provided

## Network policies

* Every pod has an ingress and egress
* Ingress - incoming request
* Egress - outgoing request
* Ingress and Egress
* So a 3 tier architecture where there is a webserver, backend, database
  * webserver 
    * Ingress: 80
    * Egress: 5000
  * API
    * Ingress: 5000
    * Egress: 3306
  * DB
    * Ingress: 3306
  * Once you allow incoming traffic,via ingress, egress is automatically allowed on the same port
  * Webserver cannot access DB directly and can access only API server
  * API server can be accessed only by webserver
  * Rules are based out of selector labels
    * podSelector specified what policy is applied to the pod
    * Ingress matching labels and namespaces allow only from those pods.
    * Egress can use ipblock selector with ports to ensure outgoing requests are forwarded to a specific ip only

## Certificates

#### Location of certificates

```shell
# List all certs
ls -ltr /etc/kubernetes/pki

# List all etcd
ls -ltr /etc/kubernetes/pki/etcd
```

#### List all certs and expiry 
```shell
kubeadm certs check-expiration
```

#### Viewing the contents of a certificate to find ALT, Issuer CN, CN

```shell
openssl x509 -in <cert-path> -text -noout
```

*Note: Almost all certificates can be located at /etc/kubernetes/pki/api-server.yaml*

# Managing certificates

1. Generate a certificate with a key
2. Create a Certificate signing request.
3. approve/deny csr