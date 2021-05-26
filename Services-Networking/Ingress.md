# Ingress Resource

An Ingress resource provides external accessto multiple services over a single load balanver.
Ingresses operate at the application layer of the network stack (HTTP) and can provide features such as 
cookie-based session affinity and the like, which services can't.

* Create the service
```
kubectl create -f kubia-rc.yaml

kubectl create -f kubia-svc-nodeport.yaml

 kubectl create -f kubia-ingress.yaml
```

* Show the service
```
kubectl get ingress --all-namespaces
```

Create a host entry:
```
192.168.86.46	kubia.example.com
```

Browse kubia.example.com

# Cleanup
```
# Delete the service
```
kubectl delelte rc kubia

kubectl delete service kubia-nodeport

kubectl delete ingress kubia
```

# Diagostics

```
kubectl get pods 

kubectl logs <podname>
```