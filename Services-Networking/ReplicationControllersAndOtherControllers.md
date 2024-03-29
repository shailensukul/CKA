# Replication Controllers And Other Controllers
[Back](./ReadMe.md)

## Pods
Pods represent the  basic deployable unit in Kubernetes.

## [Replication Controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)
A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.

### Liveness Probe
Kubernetes periodically checks the liveiness probe and restarts the container if the probe fails.

3 mechanisms of liveness probes:
* HTTP GET Probe - a HTTP GET is performed on the container's IP address, a port and path that you specify.
If the probe returns a sucess response (2xx or 3xx), it is considered successful.
Otherwise, it is considered a failure and the container will be restarted.

```
apiVersion: v1
kind: Pod
metadata:
    name: kubia-liveness
spec:
    containers:
    - image: luksa/kubia-unhealthy
      name: kubia
      livenessProbe:
        httpGet:
            path: /
            port: 8080
```

* TCP Socket Probe - tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful. Otherwise, the container is restarted.

* Exec Probe - executes an arbitary command inside a container and checks the command's exit code. If the status code is 0, the probe is sucessful. All other codes are considered failures.

### Creating an HTTP-based liveness probe

```
apiVersion: v1
kind: Pod
metadata: 
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:
      httpGet:
        path: /
        port: 8080
        initialDelaySeconds:  // wait 15 seconds before executing the first probe
```

### Seeing a liveness probe in action
```
kubectl get pod kubia-liveness
```

```
NAME            READY   STATUS    RESTARTS  AGE
kubia-liveness  1/1     Running   1         2m
```

The RESTARTS column shows that the container has been restarted once.

To obtain the application log of the previous crashed container:

```
kubectl logs myprod --previous
```

You can also get the reason for the container restart by:

```
kubectl describe pod kubia-liveness
```

You will get the exit code from the `describe` command above. For example, 137 signals that the process was killed by an external signal (exit code is 128 
+ 9 (SIGKILL). Likewise, exit code 143 corresponds to 128 + 15 (SIGTERM).

The Kubelet on the node hosting the Pod performs the liveness probe and restarts the container, if required. However, if the node crashes, the Control Plane restarts the node and recreates the Pods. 

It will not recreate the Pods, if they were created directly. To ensure it knows about them, you have to use a ReplicationController.

## ReplicationController

A replication controller is a Kubernetes resource which ensures that pods are running by using a label selector.

## Replication Controller Logic

![Replication Controller Logic](/images/Services-ReplicationControllerLogic.jpg)

It has 3 parts:
* A label selector - determines what pods are in the ReplicationController's scope
* Replica count - specifies the desired number of pods
* Pod template - is used when creating new pod replicas

![Replication Controller](/images/Services-ReplicationController.jpg)

<pre>A YAML definition of a Replication Controller</pre>
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector: 
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        ports:
        - containerPort: 8080 

```

### Replication controller commands

`kubectl create -f kubia-rc.yaml`

`kubectl get rc`

```
NAME    DESIRED   CURRENT   READY   AGE
kubia   3         3         3       143d
```

```
# list details of replication controller
kubectl describe rc kubia
```

```
Name:         kubia
Namespace:    default
Selector:     app=kubia
Labels:       app=kubia
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kubia
  Containers:
   kubia:
    Image:        luksa/kubia:arm
    Port:         8080/TCP
    Host Port:    0/TCP
    Readiness:    exec [ls /var/ready] delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
```


### List Nodes and Pods

```
kubectl get pods --all-namespaces -o wide
```

### List Pods on a Specific Node
```
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=pi-2
```

## Find ReplicationController for Pod

```
kubectl describe pod <podname>
```

And inspect the `Controlled By` field

```
root@pi-1:/home/ubuntu# kubectl describe pod kubia-rv95b
Name:         kubia-rv95b
Namespace:    default
Priority:     0
Node:         pi-3/192.168.86.45
Start Time:   Sun, 08 Aug 2021 11:28:28 +0000
Labels:       app=kubia
Annotations:  <none>
Status:       Running
IP:           10.42.1.19
IPs:
  IP:           10.42.1.19
Controlled By:  ReplicationController/kubia
```

## Label Pods

```
kubectl label pod kubia-rv95b type=special
```

```
kubia get pods --show-labels
```

```
kubia-mjv8z                      1/1     Running   0          90d    app=kubia
kubia-gvsnw                      1/1     Running   0          90d    app=kubia
kubia-rv95b                      1/1     Running   0          90d    app=kubia,type=special
```

## Change label of a managed pod
```
kubectl label pod kubia-rv95b app=foo --overwrite
```

## Changing the Replication Controller's Pod Template

Changing a ReplicationController's pod template only affects pods created afterward and has no effect on existing pods.

```
## Edit ReplicationController
kubectl edit rc kubia
```

Find the pod template section and add an additional label to the metadata. 
```
spec:
  replicas: 3
  selector:
    app: kubia
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: kubia
        app: kubia2 # add another label
```

```
## Delete an existing pods
kubectl delete pod kubia-frwkl
```

```
kubectl get pods --show-labels
```

## Edit the Replication Controller
```
kubectl edit rc kubia
```

## Delete Replication Controller  
```
# Delete the replication controller but leave the pods
kubectl delete rc kubia --cascade=false
```