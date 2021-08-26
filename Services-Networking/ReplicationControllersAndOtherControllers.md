# Replication Controllers And Other Controllers
[Back](./ReadMe.md)

Pods - represent the  basic deployable unit in Kubernetes.

## Livelness Probe
Kubernetes periodically checks the liveiness probe and restarts the container if the probe fails.

3 mechanisms of liveness probes:
* HTTP GET Probe - a HTTP GET is performed on the container's IP address, a port and path that you specify.
If the probe returns a sucess response (2xx or 3xx), it is considered successful.
Otherwise, it is considered a failure and the container will be restarted.

```
apiVersion: v1
king: Pod
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

* Exec Probe - executes an arbitary command inside a container and checks the command's exist code. If the status code is 0, the probe is sucessful. All other codes are considered failures.

