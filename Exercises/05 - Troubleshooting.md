[Back](../README.md)

Lab Exercises for Troubleshooting
=================================

Exercise 0 - Setup
==================
-   Prepare a cluster (Single node, kubeadm, k3s, etc)
-   Open browser tabs to <https://kubernetes.io/docs/>, <https://github.com/kubernetes/> and <https://kubernetes.io/blog/> (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))
-   Ensure your etcd nodes have `etcdctl` installed

Exercise 1 - Evaluate cluster and node logging
==============================================

1.  For your cluster type, determine how to acquire logs for your master nodes. They could be in the form of:
    1.  Services
    2.  Static Pods
    3.  Kubernetes Pods
2.  Using `kubectl` get a list of `events` from your cluster
3.  Using `etcdctl` determine the health of the etcd cluster

## Answer

1.  1 Is dependent on how the cluster was made and potentially which OS's were used. For example, if K8S components manifest as Kubernetes Pods:

```source-shell
kubectl logs <podname> <namespace>
```

```source-shell
kubectl get events
```

```source-shell
etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status
+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|        ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://127.0.0.1:2379 | 4e30a295f2c3c1a4 |   3.5.0 |  8.1 MB |      true |      false |         3 |       7903 |               7903 |        |
+------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

```

Exercise 2 - Understand how to monitor applications
====================================================

1.  Deploy the following manifest : 
```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox:1.28
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
```
2.  Acquire the logs from this pod - what do they mention?

## Answer

```source-shell
kubectl logs counter
0: Sun Feb 14 19:09:01 UTC 2021
1: Sun Feb 14 19:09:02 UTC 2021
2: Sun Feb 14 19:09:03 UTC 2021
3: Sun Feb 14 19:09:04 UTC 202
```

Exercise 3 - Troubleshoot application failure
=============================================

1.  Deploy the following manifest: 
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-configmap
      image: nginx:invalidversion
``` 
It will create a Pod named `nginx-pod`. Determine why this Pod will not enter `running` state by using `kubectl`

Answer

Exercise 4 - Troubleshoot networking
=====================================

1.  Deploy the following manifest: 
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
    targetPort: 8080
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

It will create `deployment` and `service` objects. Identify the DNS name of this service.
2.  Test resolution of this DNS record:
    1.  Create a Pod that has `nslookup` installed. ie: `kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml`
3.  Test sending traffic to this service
    1.  Create a Pod that has `curl` installed. ie: `kubectl run curl --image=radial/busyboxplus:curl -i --tty` - Why does it fail?
4.  Rectify the identified error in step 3

## Answer

```source-shell
nginx-service.default.svc.cluster.local
```

```source-shell
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
kubectl exec -it dnsutils sh
/ # nslookup nginx-service.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	nginx-service.default.svc.cluster.local
Address: 10.99.41.254
```

```source-shell
kubectl run curl --image=radial/busyboxplus:curl -i --tty
If you don't see a command prompt, try pressing enter.
[ root@curl:/ ]$ curl nginx-service.default.svc.cluster.local
curl: (7) Failed to connect to nginx-service.default.svc.cluster.local port 80: Connection refused
```

Check service:

```source-shell
kubectl describe service nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Families:       <none>
IP:                10.99.41.254
IPs:               10.99.41.254
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.1.18:8080,10.244.1.19:8080,10.244.1.20:8080
Session Affinity:  None
Events:            <none>
```

Note:

-   Service is listening on port 80
-   Service has a endpoint list, with target port of 8080
-   Test `curl` directly against pod:

```source-shell
curl 10.244.1.18:8080
curl: (7) Failed to connect to 10.244.1.18 port 8080: Connection refused
```

Port 8080 isn't listening, check the pod config:

```source-shell
kubectl describe po nginx-deployment-5d59d67564-bk9xb | grep -i "port:"
    Port:           80/TCP
```

The service is trying to forward traffic to port 8080 on the container, but the container is only listening on port 80. Reconfigure the `service` object, ie:

```source-shell
kubectl edit service nginx-service
Replace targetPort: 8080 with targetPort: 80
```

Retest:

```source-shell
 curl nginx-service.default.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```
