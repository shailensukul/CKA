# Services

Service - is a resource which provides a single, constant point of entry to a group of pods  providing the same service.

* Create the service
```
 kubectl create -f kubia-svc.yaml
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