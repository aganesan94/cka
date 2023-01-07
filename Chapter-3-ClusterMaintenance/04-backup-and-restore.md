# Backup Candidates

There are 3 things to back up

1. Resource configuration 
2. ETC Cluster
3. Any persistent store

## Resource Configuration

* To be stored in github.
* API server can be queried with a command such as 

```
# Backing up all resources in a namespace
kubectl get all --all-namespaces -o yaml > <filename>
```

## ETCD

* etcd can save snapshot by using its own cli.
* *--data-dir* argument used when setting up etcd. This is used for backing up the data
* Using the etcdcli to take a snapshot of etcd.
* Snapshots are to be created after stopping the kube-api-server
* Restore the snapshot and specify the *--data-dir*

etcdctl is a command line client for etcd.

In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3. To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.

You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:

```
export ETCDCTL_API=3
```

For example, if you want to take a snapshot of etcd, use:

```shell
# -h and keep a note of the mandatory global options.
ETCDCTL_API=3 etcdctl snapshot save <databasefile>

# Snapshot status can be viewed by the status command
ETCDCTL_API=3 etcdctl snapshot status

# Restoring etcd after stopping kube-apiserver
systemctl stop kube-apiserver
ETCDCTL_API=3 etcdctl snapshot restore <databasefile> --data-dir <dir-to-store>

# Restart services
systemctl daemon-reload
service etcd restart
```