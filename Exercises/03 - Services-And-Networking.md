[Back](../README.md)

Lab Exercises for Services & Networking
=======================================

Exercise 0 - Setup
==================

-   Prepare a cluster (Single node, kubeadm, k3s, etc)
-   Open browser tabs to <https://kubernetes.io/docs/>, <https://github.com/kubernetes/> and <https://kubernetes.io/blog/> (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))

[](https://github.com/David-VTUK/CKA-StudyGuide/blob/master/LabGuide/03-Services%20%26%20Networking.md#exercise-1---understand-connectivity-between-pods)

Exercise 1 - Understand connectivity between Pods
=================================================

1.  Deploy the following manifest
```
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
2.  Using `kubectl`, identify the Pod IP addresses
3.  Determine the DNS name of the service.

## Answer

Identify the `selector` for the service:

```source-shell
kubectl describe service nginx-service | grep -i selector
Selector:          app=nginx
```

Filter `kubectl` output:

```source-shell
kubectl get po -l app=nginx -o wide
```

Service name will be, based on the format `[Service Name].[Namespace].[Type].[Base Domain Name]` :
You can run an interactive shell in your pod and run the nslookup command to get the FQDN
```
kubectl run myshell -it --rm --restart=Never --image=registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3 -- /bin/sh

```

```source-shell
nginx-service.default.svc.cluster.local
```

Exercise 2 - Understand ClusterIP, NodePort, LoadBalancer service types and endpoint
====================================================================================

1.  Create three `deployments` of your choosing
2.  Expose one of these deployments with a service of type `ClusterIP`
3.  Expose one of these deployments with a service of type `Nodeport`
4.  Expose one of these deployments with a service of type `Loadbalancer`
    1.  Note, this remains in `pending` status unless your cluster has integration with a cloud provider that provisions one for you (ie AWS ELB), or you have a software implementation such as `metallb`

## Answer

```source-shell
kubectl create deployment nginx-clusterip --image=nginx --replicas 1
kubectl create deployment nginx-nodeport --image=nginx --replicas 1
kubectl create deployment nginx-loadbalancer --image=nginx --replicas 1
```

```source-shell
kubectl expose deployment nginx-clusterip --type="ClusterIP" --port="80"
kubectl expose deployment nginx-nodeport --type="NodePort" --port="80"
kubectl expose deployment nginx-loadbalancer --type="LoadBalancer" --port="80"
```

Exercise 3 - Know how to use Ingress controllers and Ingress resources
======================================================================

1.  Create an `ingress` object named `myingress` with the following specification:

-   Manages the host `myingress.mydomain`
-   Traffic to the base path `/` will be forwarded to a `service` called `main` on port 80
-   Traffic to the path `/api` will be forwarded to a `service` called `api` on port 8080

## Answer 

```
kubectl create ingress myingress --rule="myingress.mydomain/=main:80" --rule="myingress.mydomain/api=api:8080"

```

Exercise 4 - Know how to configure and use CoreDNS
====================================================

1.  Identify the configuration location of `coredns`
2.  Modify the coredns config file so DNS queries not resolved by itself are forwarded to the DNS server `8.8.8.8`
3.  Validate the changes you have made
4.  Add additional configuration so that all DNS queries for `custom.local` are forwarded to the resolver `10.5.4.223`

## Answer
```source-shell
kubectl get cm coredns -n kube-system
NAME      DATA   AGE
coredns   2      94d
```

```source-shell
kubectl edit cm coredns -n kube-system

replace:
forward . /etc/resolv.conf

with
forward . 8.8.8.8
```

Add the block:

```source-shell
custom.local:53 {
        errors
        cache 30
        forward . 10.5.4.223
        reload
    }
```