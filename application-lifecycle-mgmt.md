# Application Lifecycle management

## Rollout strategy

* Search for rollout in cheatsheet https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/
* Strategies
  * Recreate: Kill all existing pods before new ones are recreated.
  * RollingUpdate (default)
    * MaxUnavailable: Default is 25%. Max pods unavailable during the rollout process
    * MaxSurge: Default is 25%. During rollout pods are scaled upto 125% (old and new pods together), once old pods are killed the scaling is still at 125% of desired pods


## Command and argument

* Any numerics inside the command should be double quoted
* An argument can exist without a command in the pod definition file
* An imperative command to create a prod with command is:

```shell
kubectl run nginx --image=nginx --dry-run=client -o yaml --command sleep 5000
```

## Environment variables/ConfigMaps/Secrets

* Learn different ways of how they can be included in the deployment
* Loading an environment variable
  * With a value
  * Config Maps/Scerets
    * Loading using *envFrom*
    * Loading specific variables using *valueFrom*
    * Using completely as a volumemount
    * Loading specific files as a volumemount (this creates a file)
* Remember the following as possible options
  * env
  * envFrom
  * configMapKeyRef
  * secretRef
  * valueFrom 

---

* [Environment](additional-resources/environment.md)
* [Secrets](additional-resources/secrets.md)
* [ConfigMaps](additional-resources/configmaps.md)
 
##  Multiple Container pods
  * Refer to section to be able to debug logs 

## Initcontainers
* If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. 
* However, if the Pod has a restartPolicy of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.
* Are run before the main container runs
* If there are multiple initcontainers, the first one is run, before the subsequent ones run in order

## Scaling
* Vertical: Increasing the size of existing server (increasing resources on existing pods)
* Horizontal: Add new servers (more pods)
* Works on deployments, replicasets and statefulsets
* resources field needs to be provided in the deployment for scaling to work.

### 2 types of scaling

* Horizontal and Vertical
   * Manual
     * kubeadm join
     * kubectl scale
     * kubectl edit and modify cpu resources
   * Automatic
     * Cluster autoscaler
     * Horizontal Pod autoscaler
     * Vertical Pod autoscaler

Search for (kubetl autoscale) in https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/

* Inplace changing of resources limit in pods is limited to cpu and memory only


