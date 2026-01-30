# Storage

## Container storage interface (CSI)

* Common interface for storage

## Persistent Volume Claims
## Persistent Volumes

The access modes are:

* ReadWriteOnce – the volume can be mounted as read-write by a single node
* ReadOnlyMany – the volume can be mounted read-only by many nodes
* ReadWriteMany – the volume can be mounted as read-write by many nodes

PersistentVolumes can have various reclaim policies, including 
* Retain
* Recycle 
* Delete. 

For dynamically provisioned PersistentVolumes, the default reclaim policy is “Delete”. 
This means that a dynamically provisioned volume is automatically deleted when a user deletes the corresponding PersistentVolumeClaim. 
This automatic behavior might be inappropriate if the volume contains precious data. 
Notice that the RECLAIM POLICY is Delete (default value), which is one of the two reclaim policies, 
the other one is Retain. (A third policy Recycle has been deprecated). 
In case of Delete, the PV is deleted automatically when the PVC is removed, 
and the data on the PVC will also be lost.

## Storage classes

* local-storage class does not allow dynamic provisioning

Reference: https://kubernetes.io/docs/concepts/storage/storage-classes/