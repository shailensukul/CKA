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

The format is: 
`<service name>_SERVICE_HOST`
`<service name>_SERVICE_PORT`

### Via DNS
The `kube-dns` pod, which lives in the `kube-system` namespac, runs a DNS server.
Kubernetes configures the `/etc/resolv.conf` for each pod so that the pod points to the Kubernetes internal DNS server.

Note: You can configure the DNS server by modifying the `dnsPolicy` property in each pod's spec.

You can use the FQDN to access the service:

`<service name>.<service namespace>.svc.cluster.local`
Ex:
`backend-database.default.svc.cluster.local`


Note: You still need to grab the port number from the environment variable.


## Running in a shell in a Pod's container

```
kubectl exec -it <podname> -- bash 
```

Now you can curl the service
```
curl http://kubia
```

or
```
curl http://kubia.default.svc.cluster.local
```

Note: You will not be able top ping a service IP because it is a virtual IP.

## Connecting to services outside the cluster
Endpoints sit in between a Service and the resource it links to.
You can get endpoints by:

`kubectl describe service kubia`

## EndPoints
*Endpoints* is a list of IP addressses and ports exposing a service.

Get endpoints:
```
kubectl get endpoints <servicename>
```

Note: If you create a service without a selector, it will not create any endpoints.

### External Service

The following service definition does not create any endpoints:
```
apiVersion: v1
kind: Service
metadata:
    name: external-service
spec:
    ports:
    - port: 80
```

Here is how to create an endpoint:

The Endpoints object needs to have the same name as the Service.
```
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets: 
  - addresses:
    - ip: 11.11.11.11
    - pi: 22.22.22.22
    ports:
      port: 80
```
![Figure 5.4: Pods consuming a service with two external endpoints](/images/Endpoints.jpg)

## ExternalName Service
```
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: someapi.somecompany.com
  ports:
  - port: 80
```

Clients can then connect to external-service.default.svc.cluster.local

ExternalName services are implemented solely at the DNS level - a CNAME DNS is record is created for the service.
Clients will connect to the external service directly, bypassing the service proxy completely.

This is why ExternalName services do not get a CLuster IP.

## Exposing Services to External Clients

![Figure 5.5: Exposing a service to external clients](/images/Services-Expose-Service.jpg)

* NodePort service type - each cluster node opens a port on the node itself and redirects traffic received on that port to the underlying service. 
Service is available in the internal cluster IP, but also a dedicated port on all nodes.

* LoadBalancer service type - makes the service available through a dedicated load balancer, provisioned from the Cloud infrastructure Kubernetes is running on.
Clients connect via the LoadBalancer IP and it redirects traffic to the node port across all nodes.

* Ingress Resource - operates at the HTTP level (network layer 7) and exposes multiple services through a single IP address

### NodePort Service
The service can be accessed by its internal IP and also through anynode's IP and the reserved node port.

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30123
  selector:
    app: kubia
```
i
`root@pi-1:/home/ubuntu# `kubectl get service kubia-nodeport`
```
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubia-nodeport   NodePort   10.43.72.114   <none>        80:30123/TCP   55d
```

The External-IP shows `<nodes>` indicating that the service is accessible through the IP address of any cluster node.
The Port(s) column shows both the internal port of the cluster IP (80) and the node port (30123).

The service is available at the following addresses:

* 10.43.72.114:80
* <1st node's IP>:30123
* <2nd node's IP>:30123

![Figure 5.6: An external client connecting to a NodePort service either through Node 1 or 2](/images/Services-NodePort.jpg)

#### Changing firewall rules to let external clients access the NodePort service

```
gcloud compute firewall-rules create kubia-svc-rule --allow=tcp:30123
```

Now you can access your nodes externally on port 30123, you need to work out the IP of the node:
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
```

### Load Balancer Service