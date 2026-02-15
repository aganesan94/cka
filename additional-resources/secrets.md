# Secrets

<!-- TOC -->
* [Secrets](#secrets)
  * [Converting data to base64](#converting-data-to-base64)
  * [Converting base64 to data](#converting-base64-to-data)
  * [Creating a secret](#creating-a-secret)
  * [Creating a secret with kubectl](#creating-a-secret-with-kubectl)
  * [Ways to access secrets](#ways-to-access-secrets)
    * [1- Via a volume - all values in the Secret file](#1--via-a-volume---all-values-in-the-secret-file)
    * [2- Mapping each secret to a specific path](#2--mapping-each-secret-to-a-specific-path)
    * [3- Specifying secrets to the permissions](#3--specifying-secrets-to-the-permissions)
    * [4- Injecting specific secrets only (without volume mount)](#4--injecting-specific-secrets-only-without-volume-mount)
    * [5- Injecting all variables of a secret (without volume mount)](#5--injecting-all-variables-of-a-secret-without-volume-mount)
<!-- TOC -->

## Converting data to base64

```shell
echo -n 'my-app' | base64
echo -n '39528$vdg7Jb' | base64
```

## Converting base64 to data

```shell
echo QWxhZGRpbjpvcGVuIHNlc2FtZQ== | base64 --decode
```

## Creating a secret

* plug base64 encoded string in

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: bXktYXBw
  password: Mzk1MjgkdmRnN0pi
```

## Creating a secret with kubectl

```shell
kubectl create secret generic test-secret --from-literal='username=my-app' --from-literal='password=39528$vdg7Jb'
```

## Ways to access secrets

### 1- Via a volume - all values in the Secret file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      volumeMounts:
        # name must match the volume name below
        - name: secret-volume
          mountPath: /etc/secret-volume
          readOnly: true
  # The secret data is exposed to Containers in the Pod through a Volume.
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret
```

### 2- Mapping each secret to a specific path

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```
### 3- Specifying secrets to the permissions

* Default permissions is 0644

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 0400
```

### 4- Injecting specific secrets only (without volume mount)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envvars-multiple-secrets
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: BACKEND_USERNAME
      valueFrom:
        secretKeyRef:
          name: backend-user
          key: backend-username
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-user
          key: db-username
```

### 5- Injecting all variables of a secret (without volume mount)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    envFrom:
    - secretRef:
        name: test-secret
```