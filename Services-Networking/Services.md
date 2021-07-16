# Services

Service - is a resource which provides a single, constant point of entry to a group of pods  providing the same service.
Each service has an IP address and port that never chnage while the service exists.

![Figure 5.1 Both internal and external clients usually connect to pods through services](/images/Services-Example.jpg)


* Create the service
```
 kubectl create -f kubia-svc.yaml
```

* Get the service
```
kubectl get service
```

* Describe the service
```
kubectl describe svc kubia
```

* Connecting to the service
```
# Get the pod
kubectl get pods

# Execute a curl on the pod shell
kubectl exec kubia-qwgfq -- curl -s http://10.99.46.75

```

Endpoints - is a list of IP addresses and ports exposing a service

```
kubectl get endpoints kubia
```

# Cleanup
```
# Delete the service
kubectl delete svc kubia

# Delete the replication controller
kubectl delete rc kubia

# Delete the pod
kubectl delete pod kubia-qwgfq
```

## Session Affinity in service
Types of affinity:
* ClientIP
* None

```
apiVersion: V1
kind: Service
spec:
    sessionAffinity: ClientIP
...
```

## Exposing Ports
```
apiVersion: v1
kind: Service
metadata:
    name: kubia
spec:
    ports:
    - name: http
      port: 80
      targerPort: 8080
    - name: https
      port:443
      targetPort: 8443
selector:
    app: kubia

```

### Using named ports
Port definition:
```
kind: Pod
spec:
    containers:
    - name: kubia
    ports:
    - name: http
      containerPort: 8080
    - name: https
      containerPort: 8443

```

You can then refer to these ports by name:
```
apiVersion: v1
kind: Service
spec:
    ports:
    - name: http
      port: 80
      targetPort: http
    - name: https
      port: 443
      targetPort: https
```

## Discovering Services

### Via Environment Variables
Get the environment variable in a pod by:

```
kubectl exec kubia-3inly env
```

Output
```
...
KUBIA_SERVICE_HOST=10.111.249.153
KUBIA_SERVICE_PORT=80
...
```


