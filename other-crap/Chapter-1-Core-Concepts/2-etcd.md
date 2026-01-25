# ETCD

https://etcd.io/

What is etcd?

* etcd is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines. 

* It gracefully handles leader elections during network partitions and can tolerate machine failure, even in the leader node

* 2379 is the default port where etcd listen

## ETCD from a K8 perspective

* All information related to the cluster is stored in etcd
* When it is setup in a cluster, each etcd should know about the other nodes in the cluster which is configured in the --initial-cluster parameter

### ETCD v2

```shell
./etcdctl
./etcdctl set key1 value1
./etcdctl get key1
./etcdctl --version
```
### ETCD v3

```shell
# Setting a particular version of ETCDCTL
export ETCDCTL_API=3
./etcdctl put key1 value1
./etcdctl get key1
```
### Provides the etcd master information

```shell
kubectl get pods -n kube-system
```

### Get information about all the keys in the registry

```shell
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```
