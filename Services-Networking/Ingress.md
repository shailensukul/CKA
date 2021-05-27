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

# TLS

## Create the private key and certificate
```
$ openssl genrsa -out tls.key 2048
$ openssl req -new -x509 -key tls.key -out tls.cert -days 360 -subj /CN=kubia.example.com
```

## Create the secret from the two files:
```
kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key
```

## Update the Ingress

```
kubectl apply -f kubia-ingress-tls.yaml
```

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