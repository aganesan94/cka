# Kube Controller Manager

*  Is a daemon that embeds the core control loops shipped with Kubernetes

* In Kubernetes, a controller is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state.
* 

## Examples

* Node Controller

  * Checks the state of the nodes every 5 seconds.
  * If a node is unreachable the controller gives the node 5 minutes to come back up. If it does not , it evicts the pods from that node and moves them to another node.

* Replica Controller

  * Replication Controller ensures the desired amount of replicas are always around.

* Other controllers

    * Deployment Controller
    * Namespace Controller
    * Endpoint Controller
    * Cronjob controller
    * Job Controller
    * PV Protection Controller
    * PV Binder Controller
    * Service Account Controller
    * Statefulset Controller
