# Services
[Back](./ReadMe.md)

An abstract way to expose an application running on a set of Pods as a network service.
Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

![Figure 5.1 Both internal and external clients usually connect to pods through services](/images/Services-Example.jpg)

Sample service file:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http
      - containerPort: 8443
        name: https
        
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: http
  - name: https
    protocol: TCP
    port: 443
    targetPort: https
```

## Discovering Services

`kubectl exec kubia-3inly env`

Shows service names and ports
```
KUBIA_SERVICE_HOST=10.111.249.153
KUBIA_SERVICE_PORT=80

KUBERNETES_SERVICE_HOST=10.111.249.153
KUBERNETES_SERVICE_POST=443
```

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
```

## Execute a curl on the pod shell
```
kubectl exec kubia-qwgfq -- curl -s http://10.99.46.75
```

## Open a bash shell in the pod
```
kubectl exec -it kubia-qwgfq -- bash
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

Note: You will not be able to ping a service IP because it is a virtual IP.

## Services without Selectors

Services most commonly abstract access to Kubernetes Pods thanks to the selector, but when used with a corresponding Endpoints object and without a selector, the Service can abstract other kinds of backends, including ones that run outside the cluster. 

For example:

* You want to have an external database cluster in production, but in your test environment you use your own databases.
* You want to point your Service to a Service in a different Namespace or on another cluster.
* You are migrating a workload to Kubernetes. While evaluating the approach, you run only a portion of your backends in Kubernetes.

In any of these scenarios you can define a Service without a Pod selector. For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Because this Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to the network address and port where it's running, by adding an Endpoints object manually:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  # the name here should match the name of the Service
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

![Figure 5.4: Pods consuming a service with two external endpoints](/images/Endpoints.jpg)

Accessing a Service without a selector works the same as if it had a selector. In the example above, traffic is routed to the single endpoint defined in the YAML: `192.0.2.42:9376` (TCP).

Endpoints sit in between a Service and the resource it links to.
You can get endpoints by:

`kubectl describe service kubia`

### EndPoints

*Endpoints* is a list of IP addressses and ports exposing a service.

Get endpoints:
```
kubectl get endpoints <servicename>
```

## Service Types

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that's outside of your cluster.
Kubernetes `ServiceTypes` allow you to specify what kind of Service you want. The default is `ClusterIP`.

`Type` values and their behaviors are:

* `ClusterIP`: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default `ServiceType`.

* `NodePort`: Exposes the Service on each Node's IP at a static port (the `NodePort`). A `ClusterIP` Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the `NodePort` Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.

* `LoadBalancer`: Exposes the Service externally using a cloud provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external load balancer routes, are automatically created.

* `ExternalName`: Maps the Service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value. No proxying of any kind is set up.


## Type NodePort

If you set the type field to `NodePort`, the Kubernetes control plane allocates a port from a range specified by `--service-node-port-range` flag (default: 30000-32767). Each node proxies that port (the same port number on every Node) into your Service. Your Service reports the allocated port in its `.spec.ports[*].nodePort` field.

Using a NodePort gives you the freedom to set up your own load balancing solution, to configure environments that are not fully supported by Kubernetes, or even to expose one or more nodes' IPs directly.

Note that this Service is visible as `<NodeIP>:spec.ports[*].nodePort` and `.spec.clusterIP:spec.ports[*].port`. If the `--nodeport-addresses` flag for kube-proxy or the equivalent field in the kube-proxy configuration file is set, `<NodeIP>` would be filtered node IP(s).

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```

## Type ExternalName Service

Services of type ExternalName map a Service to a DNS name, not to a typical selector such as my-service or cassandra. You specify these Services with the spec.externalName parameter.

This Service definition, for example, maps the my-service Service in the prod namespace to my.database.example.com

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

![Figure 5.5: Exposing a service to external clients](/images/Services-Expose-Service.jpg)

When looking up the host `my-service.prod.svc.cluster.local`, the cluster DNS Service returns a `CNAME` record with the value `my.database.example.com`. 
Accessing `my-service` works in the same way as other Services but with the crucial difference that redirection happens at the DNS level rather than via proxying or forwarding. 

Should you later decide to move your database into your cluster, you can start its Pods, add appropriate selectors or endpoints, and change the Service's type.

> **Warning:**
>
> You may have trouble using ExternalName for some common protocols, including HTTP and HTTPS. If you use ExternalName then the hostname used by clients inside your cluster is different from the name that > the ExternalName references.
>
> For protocols that use hostnames this difference may lead to errors or unexpected responses. HTTP requests will have a `Host:` header that the origin server does not recognize; TLS servers will not be  able to provide a certificate matching the hostname that the client connected to.

## External IPs

If there are external IPs that route to one or more cluster nodes, Kubernetes Services can be exposed on those `externalIPs`. 
Traffic that ingresses into the cluster with the external IP (as destination IP), on the Service port, will be routed to one of the Service endpoints. externalIPs are not managed by Kubernetes and are the responsibility of the cluster administrator.

In the Service spec, `externalIPs` can be specified along with any of the ServiceTypes. 
In the example below, "`my-service`" can be accessed by clients on "`80.11.12.10:80`" (`externalIP:port`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```


### Load Balancer Service

If you set a service's type to LoadBalancer, cloud providers will automatically provision a load balancer.
If Kubernetes is running in an environment which does not support the Load Balancer service, the service will behave like a NodePort service.

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app:kubia
```
To get the public IP of the loadbalancer: 

`kubectl get svc kubia-loadbalancer`

Note: When hitting the external url via `curl http://130.211.53.173`, the browser will hit the exact same pod every time.
Using `kubectl describe kubia-loadbalancer` you can see that the session affinity is set to None. 
Because the broser is using keep-alive connections, it sends its request through a single connection, whereas curl opens a new connection every time.
When a connection is first opened, a random port is selected, and all network packets belonging to that connection are all sent to the same pod.

![Figure 5.7 An external client connecting to a LoadBalancer service](/images/Services-LoadBalancer.jpg)

External connections and network hops
When an external client connects to a service through the nodeport (even when it goes via the loadbalancer first), the random chosen pod may or may not be running on the same node that received the connection.

An additional network hop may be required to reach the node but this may not always be desirable.
The extra hop can be prevented by the following in the service spec:
```
spec:
  externalTrafficPolicy: Local
```

This will cause the node receiving the connection to pick a locally runing node. 
If there are no local noddes, then the connectiom will hang).

This also affects load balancing as it unevenly distributes connections.

Beware of ClientIP
When pods receive a connection via the NodePort, the packet's sourceIP is changed, because Source Network Address Translation (SNAT) is performed on the packets.

The backing pods cannot see the client IP. `externalTrafficPolicy: Local` will preserve the clientIP because it prevents network hops.

### Ingress Resource
Ingresses operate at the application layer of the network stack (HTTP) and direct requests to services by inspecting the host and path in the request. In this way, unlike a Load Balancer service, it can use only a single public IP address.

Ingresses operate at the application layer of the network stack (HTTP) and can provide features such as cookie-based session affinity and the like, which services can't.

Ingresses require an Ingress controller to be running in the cluster.

![Exposing multiple services through a single ingress](/images/Services-Ingress.jpg)

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com
    http:
      path:
      - path: /
        backend:
          serviceName: kubia-nodeport
          servicePort: 80
```

Create the Ingress controller
`kubectl create -f kubia-ingress.yaml`

Show Ingresses
`kubectl get ingresses`

```
NAME    CLASS    HOSTS               ADDRESS         PORTS     AGE
kubia   <none>   kubia.example.com   192.168.86.36   80, 443   61d
```

You can then edit the hosts file:
`192.168.86.36 kubia.example.com`

![Accessing pods through an ingress](/images/Services-Ingress-Pod-Access.jpg)

### Exposing multiple services through the same ingress
You can map multiple paths on the same host to different services:

```
...
- host: kubia.example.com
http:
  paths:
  - path: /kubia
  backend:
    serviceName: kubia # Requests to kubia.example.com/kubia will be routed to the kubia service
    servicePort: 80
  - path: /bar
  backend:
    serviceName: bar # Requests to kubia.example.com/var will be routed to the bat service
    servicePort: 80
```

### Configuring Ingress to handle TLS traffic

* Ingress controller terminates the TLS connection
* Application running in the Pod does not need to support TLS
* Need to attach a certificate and a private key to the Ingress, which are stored in a Kubernetes resource called a Secret

Create private key and certificate
```
openssl genrsa -out tls.key 2048
``` 

```
openssl req -new -x509 -key tls.key -out tls.cert -days 360 =subj /CN=kubia.example.com
```

Then you create the Secret from the two files:
```
kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key 
```

```
kubectl create secret tls -h
```

```
Create a TLS secret from the given public/private key pair.

 The public/private key pair must exist before hand. The public key certificate must be .PEM encoded and match the given
private key.

Examples:
  # Create a new TLS secret named tls-secret with the given key pair:
  kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --append-hash=false: Append a hash of the secret to its name.
      --cert='': Path to PEM encoded public key certificate.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --field-manager='kubectl-create': Name of the manager used to track field ownership.
      --key='': Path to private key associated with given certificate.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
      --save-config=false: If true, the configuration of current object will be saved in its annotation. Otherwise, the
annotation will be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].
      --validate=true: If true, use a schema to validate the input before sending it

Usage:
  kubectl create secret tls NAME --cert=path/to/cert/file --key=path/to/key/file [--dry-run=server|client|none]
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

#### CertificateSigningRequest resource

You can also create a regular certificate request, put it into a CSR, then have the request approved by a human or automated process.
```
kubectl certificate approve <name of CSR>
```

The signed certificate can then be retrieved from the CSR's `status.certificate` field.
Note: The certificate signer component must be running in the cluster

#### Apply to Ingress
Update your Ingress so it will accept HTTPS requests:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  tls:
  - hosts:
    - kubia.example.com
    secretName: tls-secret
rules:
- host: kubia.example.com
http:
  paths:
  - path: /
    backend:
      serviceName: kubia-nodeport
      servicePort: 80
```

### Readiness Probes

The readiness probe is invoked periodically and determines whether the specific pods should receive client requests or not.

Types of readiness probes:

* Exec probe - process is executed and the container's status is determined by the return code
* HTTP GET probe - sends HTTP GET request to the container and the HTTP status code determines readiness
* TCP socket probe - opens a TCP connection to a specified port of the container. If the connection is established, the container is considered ready.

If a pod reports that it is not ready, it is removed from the service.
If a pod then becomes ready, it is re-added.

Unlike a liveness probe, the container is not killed and restarted.

![Figure 5.11 A pod whose readiness probe fails is removed as an endpoint of a service](/images/Services-Pod-ReadinessProbe.jpg)

Figure 5.11 A pod whose readiness probe fails is removed as an endpoint of a service

#### Adding a readiness probe to a pod
Edit the pod's replication controller:

```
apiVersion: v1
...
spec:
...
  template:
    ...
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        readinessProbe:
          exec:
            command:
            - ls
            - /var/ready
        ...
```

The readiness probe will periodically perform `ls /var/ready` inside the container. The `ls` command 
returns exit code zero if the file exists, or a non-zero exit code otherwise. If the file exists, the readiness probe
will succeed, otherwise it will fail.


Delete existing Pods and watch the `Ready` of the recreated Pods.

`kubectl get pods`

```
NAME                             READY   STATUS    RESTARTS   AGE
kubia-mjv8z                      0/1     Running   0          4m35s
kubia-rv95b                      0/1     Running   0          4m35s
kubia-gvsnw                      0/1     Running   0          4m35s
```

Now simulate the readiness of one of the Pods

`kubectl exec kubia-2r1qb -- touch /var/ready`

```
NAME                             READY   STATUS    RESTARTS   AGE
kubia-mjv8z                      0/1     Running   0          5m25s
kubia-gvsnw                      0/1     Running   0          5m25s
kubia-rv95b                      1/1     Running   0          5m25s
```

To get more details about the readiness of a Pod, use
```
kubectl describe pod kubia-gvsnw
```

You can watch the active endpoints by : 
```
kubectl get endpoints kubia-nodeport
```

## Using a headless service for discovering individual Pods

Scenario: What if a client needs to connect to all pods, or if the backiing pods themselves need to connect to all other backing pods.

You can set the service `clusterIP` field to `None` in the service specification. 

Then the DNS looup will return pod IPs, instead of the single service IP.

### Creating a headless service

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-headless
spec:
  clusterIP: None // this makes the service headless
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubia
```

`kubectl get svc`

```
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubia-headless             ClusterIP      None           <none>        80/TCP         7s
```

### Performing a lookup
You can directly create a Pod with the network tools to perform a lookup

```
kubectl run --image=praqma/network-multitool dnsutils --command -- sleep infinity
```

Then use the pod to look up the headless service:

`kubectl exec dnsutils -- nslookup kubia-headless`

```
Server:         10.43.0.10
Address:        10.43.0.10#53

Name:   kubia-headless.default.svc.cluster.local
Address: 10.42.2.17
Name:   kubia-headless.default.svc.cluster.local
Address: 10.42.1.19
Name:   kubia-headless.default.svc.cluster.local
Address: 10.42.1.18
```

Cross reference that with the pod ips:

` kubectl get pods -o wide`

```
NAME                             READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
kubia-rv95b                      1/1     Running   0          85m     10.42.1.19   pi-3     <none>           <none>
kubia-mjv8z                      1/1     Running   0          85m     10.42.1.18   pi-3     <none>           <none>
kubia-gvsnw                      1/1     Running   0          85m     10.42.2.17   pi-2     <none>           <none>
```

* Client can still to a headless service's pods, but the load balancing will be round robin.
* Clients connect directly to the pods instead of the service proxy.

### Discovering all pods - even those that aren't ready
Add the following to the service definition:

```
kind: Service
spec:
  publishNotReadyAddresses: true
```

## Troubleshooting Services

* Make sure that you are connecting to the service's cluster IP from within the cluster and not outside it
* Do not ping the service IP as it is a virutal and will not respond to ping
* If you've defined a readiness probe, make sure that it is succeeding, otherwise the pod will not be part of the service
* Examine the endpoints to confirm that the pod is a part of the service: `kubectl get endpoints <serviceName>`
* If the service FQDN is not responding, try using the cluster IP
* Check whether you are connecting to the service port and not the target port
* Try connecting to the pod IP directly to confirm your pod is accepting connections on the correct port
* If you cannot access the app via the pod's direct IP, make sure that your app is not only binding to localhost

