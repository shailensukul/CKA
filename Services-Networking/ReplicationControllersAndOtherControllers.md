# Replication Controllers And Other Controllers
[Back](./ReadMe.md)

## Pods
Pods represent the  basic deployable unit in Kubernetes.

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

### Creating an HTTP-based liveliness probe

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
        initialDelaySeconds: 15
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
A replication controller is a Kubernetes resource whichensures that pods are running by using a label selector.
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