# Labs

Helpful scripts for the KodeKloud labs

_Make Nano the default editor_
`select-editor`

_Set the Kube Editor_
`export KUBE_EDITOR=nano`
`echo $KUBE_EDITOR`

_Linix Process Commands_

`-e` - display all processes
`-f` - full format - detailed listing
`ps -ef | grep kubelet`

_Vi Editor Commands_

```
vi filename

// Visual mode (to select text)
press v

// insert mode
press i

// command mode
// save and exit
:wq

// quit without saving changes
:q!

```

_If unsure about a command_
`kubectl expain replicaset`

\*Get all Kuberentes object
`kubectl get all`

_Create a pod directly_
`kubectl run podName --image=image [--dry-run=client] [-o yaml] > myfile.yaml`

_Run a Pod with arguments_
`kubectl run podName --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > myfile.yaml`

_Create from file_
`kubectl create -f myfile.yaml`

_If you want to delete and recreate_
`kubectl --force -f myfile.yaml`

_Apply changed file to an already running pod_
`kubectl apply -f myfile.yaml`

## Services

Types

- NodePort
- ClusterIP
- Load Balancer

_Change the image of a container in a pod_
`kubectl set image pod/redis redis=redis`

_Get extended information about a Pod_
`kubectl get pods -o wide --watch`

_Get pods by label_
`kubectl get pods --selector app=App1`

_Count rows_
`kubectl get pods --selector app=App1 --no-headers | wc -l`

_Create ReplicaSet file via a Deployment_
`kubectl get rs my-rs -o yaml > rs.yaml`

_ReplicaSet commands_

`kubectl create -f relicaset.yaml`

`kubectl get replicaset`

_Delete ReplicaSet_
`kubectl delete replicaset my-rs`

_Update ReplicaSet_
`kubectl replace -f replicaset.yaml`

_Scale ReplicaSet (does not change the replica in the file)_
`kubectl scale --replicas=6 -f replicaset.yaml`

_Edit ReplicaSet to scale_
`kubectl edit rs my-rs`

## Deployments

\*Create a sample Deployment file
`kubectl create  deployment mydeployment --image=nginx --replicas=5 --dry-run=client -o yaml `

\*Get deployments
`kubectl get deployments`

## Services

_Get help for creating a service_
`kubectl create service nodeport --help`

_Create a NodePort service_
`kubectl create service nodeport myservice --tcp=8080:80 --node-port=30000 --dry-run=client -o yaml`

## Imperative Commands

_Create an run a pod_
`kubectl run --image=nginx nginx`

_Create a deployment_
`kubectl create deployment --image=nginx nginx`

_Expose a port of the deployment_
`kubectl expose deployment nginx --port 80`

_Edit the in-memory Kubernetes file_
`kubectl edit deployment nginx`

_Scale the deployment_
`kubectl scale deployment nginx --replicas=5`

_Change the deployment image_
`kubectl set image deployment nginx nginx=nginx:1.18`

## Scheduling

You can control the node that a pod gets assigned to, at creation time with the `nodeName` element under `spec`

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  nodeName: node01
  containers:
  - image: ngix
    name: nginx
    resources: {}
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"ue
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

_You can add annotations to the metadata section_

```
kind: Deployment
metadata:
  creationTimestamp: null
  annotations:
    buildVersion: 1.2.3
  labels:
    app: myd
  name: myd
```

## Node

_Taint a node_
`kubectl taint nodes mynode key=value:taint-effect`

_Check for taints_
`kubectl describe node node01 | grep taints`

## Node Selectoes

_Label a node_
`kubectl label node node01 labelKey=labelValue`

_Add nodeSelector on Pod to select bode_

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: myPod
  name: myPod
spec:
  containers:
  - image: nginx
    name: myPod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeSelector:
    size: Large
status: {}
```

## Node Affinity

Search for node affinity and copy and paste the definition in the pod or deployment yaml

## Resource Limits

_Define Pod resource limits_

Request - how much does the Pod need to be scheduled (minimum required)
Limit - how much is the Pod allowed to consume (upper bound)

| Request     | Limit     | Result                                                                                           |
| ----------- | --------- | ------------------------------------------------------------------------------------------------ |
| No Requests | No Limit  | Any Pod can take up all the resources of the node and other Pods can be scheduled and be starved |
| No Requests | Limits    | Requests = Limits.                                                                               |
| Requests    | Limits    | Pods will be scheduled and limited. Pods will be limited even though resources may be available  |
| Requests    | No Limits | Each Pod will be scheduled correct and can still use resources efficiently                       |

Pod definition

```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

_Define Limit Ranges_
A policy to constract the resource allocations (limits and requests) for each object (ex Pod) in a namespace

_CPU Default Limits_

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

_Memory Default Limits_

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

`kubectl apply -f memory-constraints.yaml --namespace=constraints-mem-example`
`kubectl get limitrange`
`kubectl describe limitrange memory-constraints`

_Resource Quotas_
Limits the aggregate (total) resource consumption per namespace

`kubectl create resourcequota`

`kubectl create resourcequota myquota --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi --dry-run=client -o yaml`

```
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myquota
spec:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
status: {}
```

## Daemonsets

Runs one copy of pod per node
Daemonsets are analogous to deployments.

`kubectl create deployment elasticsearch -n kube-system --image=nginx --dry-run=client -o yaml > myds.yaml`

- Change the `kind` to DaemonSet
- Get rid of `replicas` and `strategy`

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

## Static Pods

Static are created directly by the kubelet and have no dependency on the kube-scheduler.
Kubernetes components themselves are static pods.
You can create a pod definition file in the static pod path directory

To check if it is a static pod, do
`kubectl get pod mypod -o yaml`
Check `OwnerReferences.kind` which should be set to `Node` vs `ReplicaSet`

How to find static pod path

_Find the --config= file_
`ps -ef | grep kubelet`

- Ex: `/var/lib/kubelet/config.yaml`

Then edit the config file and look for `staticPodPath:`
`nano /var/lib/kubelet/config.yaml`

Restart the kubelet to apply
`systemctl restart kubelet`

_View static pods_
`docker ps`

Kubectl will also list static pods
`kubectl get pods`

\*How to get into a node to delete a static Pod`

_First, get the node information_
`kubectl get nodes -o wide`
Copy the internal-ip

_SSH into the node_
`ssh nodeIP or nodeName`

## Schedulers

Deploy a custom scheduler as a Pod
`my-custom-scheduler.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-custom-scheduler-as-kube-scheduler
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config.yaml

    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

`my-scheduler-config.yaml`

```
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedlerName: my-schedule
leaderElection:
  leaderElect: true
  resourceNamespace: kube-system
  resourceName: lock-object-my-scheduler
```

_Use scheduler in Pod_

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name:: nginx
  schedulerName: my-customer-scheduler
```

_To view scheduler, look at events_

```
kubectl get events -o wide
```

_To view scheduler logs_
`kubectl logs my-customer-scheduler --namespace=kube-system

# Application Lifecycle Management

## Rolling Updates

Rollout status

```
kubectl rollout status deployment/mydeployment
```

Rollout revision history

```
kubectl rollout history deployment/mydeployment
```

Deployments

- Create deployment
  `kubectl create -f deployment.yaml`

- Get deployments
  `kubectl get deployments`

- Update the deployment
  `kubectl apply -f deployment.yaml`
  or
  `kubectl set image deploymeny/mydeployment nginx-container=nginx:1.9.1`

- Describe the deployment
  `kubectl describe deployment mydeployment`

- Rollback the deployment
  `kubectl rollout undo deployment/mydeployment`

## Debug Commands for Pods

```
apiVersion: v1
kind: Pod
metadata
  name: ubutnu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"] \\<== equivalent to entrypoint in Docker
    args: ["10"] \\<== equivalent to the Docker cmd args
```

`kubetl create -f pod.yaml`

## ConfigMaps

`kubectl create configmap app.config --from-literal=APP_COLOR=blue --from-literal=ENV=PROD`

`kubectl create configmap app.config --from-file=app_config.properties`

`kubetl get configmaps`

`kubectl describe configmaps`

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: myPod
  name: myPod
spec:
  containers:
  - image: nginx
    name: myPod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  envFrom:
    - configMapRef:
      name: app.config
      key: APP_COLOR (optional)

```

## Secrets

`kubectl get secrets mysecret`

`kubectl create secret generic mysecret --from-literalkey=value`

`kubectl create secret generic mysecret --from-file=filePath`

File secrets need to be encoded
`echo -n 'mypassword' | base64`

Decode
`echo -n 'encodedPassword' | base64 --decode`

```
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  extra: YmFyCg==
```

Use in pod

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
    envFrom:
    - secretRef:
        name: appSecret
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: true
```

## Scaling

Resource usage
`kubectl top pod my-pod`

### Cluster autosaler

### Horizontal Pod Autoscaler

-- manual scaling
`kubectl scale deployment my-deployment --replicas=3`

-- auto scale
`kubectl autoscale deployment my-deployment --cpu-percent=50 --min=1 --max=10`

--get status
`kubectl get hpa`

-- delete
`kubectl delete hpa my-deployment`

### Vertical Pod Autoscaler

-- manual scaling
`kubectl edit deployment`

-- needs to be installed
`kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yml`

-- check
`kubectl get pods -n kube-system | grep vpa`

`kubectl describe vpa my-vpa`
