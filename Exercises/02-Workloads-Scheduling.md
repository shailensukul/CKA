[Back](../README.md)

# Setup
Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and https://kubernetes.io/blog/ (these are permitted as per the current guidelines)

Exercise 1 - Understand deployments and how to perform rolling update and rollbacks
===========================================


1. Create a deployment object consisting of 6 pods, each containing a single nginx:1.19.5 container
2. Identify the update strategy employed by this deployment
3. Modify the update strategy so maxSurge is equal to 50% and maxUnavailable is equal to 50%
4. Perform a rolling update to this deployment so that the image gets updated to nginx1.19.6
5. Undo the most recent change

## Strategy
* Browse to https://kubernetes.io/docs
* Search for kubectl create deployment
* Click on the cheatsheet option
* Search for `create deployment`

# Answer

```
kubectl create deployment nginx --image=nginx:1.19.5 --replicas=6
kubectl describe deployment nginx (unless specified, will be listed as "UpdateStrategy")
kubectl patch deployment nginx -p "{\"spec\": {\"strategy\": {\"rollingUpdate\": { \"maxSurge\":\"50%\"}}}}" --record=true
kubectl patch deployment nginx -p "{\"spec\": {\"strategy\": {\"rollingUpdate\": { \"maxUnavailable\":\"50%\"}}}}" --record=true
kubectl set image deployment nginx nginx=nginx:1.19.6 --record=true
kubectl rollout undo deployment/nginx
```

Exercise 2 - Use ConfigMaps to configure applications
===========================================

1.  Create a configmap named `mycm` that has the following key=value pair
    1.  `key` = owner
    2.  `value` = yourname
2.  Create a pod of your choice, such as `nginx`. Configure this Pod so that the underlying container has the environent varibale `OWNER` set to the value of this configmap

# Answer

Create configmap:

```source-shell
kubectl create configmap mycm --from-literal=owner=david
```

Define Pod:

```source-shell
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
    - name: nginx-configmap
      image: nginx
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: OWNER
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mycm
              # Specify the key associated with the value
              key: owner
```

Validate:

```source-shell
kubectl logs nginx-configmap | grep OWNER
OWNER=david
```


Exercise 3 - Use Secrets to configure applications
===========================================

1.  Create a secret named `mysecret` that has the following key=value pair
    1.  `dbuser` = MyDatabaseUser
    2.  `dbpassword` = MyDatabasePassword
2.  Create a pod of your choice, such as `nginx`. Configure this Pod so that the underlying container has the the following environment variables set:
3.  `DBUSER` from secret key `dbuser`
4.  `DBPASS` from secret key `dbpassword`

# Answer
```source-shell
kubectl create secret generic mysecret --from-literal=dbuser="MyDatabaseUser" --from-literal=dbpassword="MyDatabasePassword"
```

Apply the following manifest:

```source-yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secret
spec:
  containers:
  - name: nginx-secret
    image: nginx
    command: [ "/bin/sh", "-c", "env" ]
    env:
      - name: dbuser
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: dbuser
      - name: dbpassword
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: dbpassword
  restartPolicy: Never
```

Which can be validated with:

```source-shell
kubectl logs nginx-secret | grep db
dbuser=MyDatabaseUser
dbpassword=MyDatabasePassword
```


Exercise 4 - Know how to scale applications
===========================================

1.  Create a deployment object consisting of 3 `pods` containing a single `nginx` container
2.  Increase this deployment size by adding two additional pods.
3.  Decrease this deployment back to the original size of 3 `pods`

# Answer
```
kubectl create deployment nginx --image=nginx --replicas=3
kubectl scale --replicas=5 deployment nginx
kubectl scale --replicas=3 deployment nginx
```

Excerise 5 - Understand how resource limits can affect Pod scheduling
=====================================================================

1.  Create a new namespace called "tenant-b-100mi"
2.  Create a memory limit of 100Mi for this namespace
3.  Create a pod with a memory request of 150Mi, ensure the limit has been set by verifying you get a error message.

# Answer

```source-shell
kubectl create ns tenant-b-100mi
```

Deploy the manifest:

```source-yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: tenant-b-memlimit
  namespace: tenant-b-100mi
spec:
  limits:
  - max:
      memory: 100Mi
    type: Container
```

Test with deploying:

```source-yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
  namespace: tenant-b-100mi
spec:
  containers:
  - name: default-mem-demo
    image: nginx
    resources:
      requests:
        memory: 150Mi
```

Which should return:

```source-shell
The Pod "default-mem-demo" is invalid: spec.containers[0].resources.requests: Invalid value: "150Mi": must be less than or equal to memory limit
```