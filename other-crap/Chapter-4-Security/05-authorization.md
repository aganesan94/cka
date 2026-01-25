# Authorization

```shell

# Get all the roles
kubectl get roles

# get rolebinding
kubectl get rolebindings

# Describing a role
kubectl describe role <role-name>

# Describing a rolebinding
kubectl describe rolebinding <role-name>

# Checking if you can access
# Operation is create/delete
# object is pods/deployments etc..
kubectl auth can-i <operation> <object>
kubectl auth can-i <operation> <object> --as -<username> --namespace <namespace>
```

![Alt text](images/authorization.png "Authorization")