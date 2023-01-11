# Security Policies

## Kubernetes Security Primitives

### General Criteria

* Root access should be disabled
* Password authentication to be disabled.
* SSH Key based authentication to be enabled.
* First line of defence is to control the access to API Server.
* Authentication - specifies who can access.
* All access to K8 cluster is managed thru kube api server.


#### Authentication - Who can access the cluster?

* Usernames & Passwords
* Username & Tokens
* Certificates
* External Authentication Providers - LDAP
* Service Accounts

* Kubernetes does not have the concept called users.
* Kube-api-server manages using the above provided authentication mechanisms

#### Authorization - What can they do after authentication is successful?

* RBAC Authorization
* ABAC Authorization
* Node Authorization
* Webhook Node

#### TLS Certificates

* The API Server interacts with the following components using the TLS Certificates

- ETCD
- Kubelet
- Kube Proxy
- Kube Scheduler
- Kube Controller Manager

### Actors

- Admins
- Developers
- End Users
- Bots