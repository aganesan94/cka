# Kube Scheduler

- The Kubernetes scheduler is a control plane process which assigns Pods to Nodes. 

- The scheduler determines which Nodes are valid placements for each Pod in the scheduling queue according to constraints and available resources. 
  
- The scheduler then ranks each valid Node and binds the Pod to a suitable Node. Multiple different schedulers may be used within a cluster;
  
-  kube-scheduler is the reference implementation.

### Process

1. Filter Nodes for what does not fit the profile
2. Rank the nodes based on a priority function and assign a scale based on resources after the pod placed.