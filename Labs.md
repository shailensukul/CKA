# Labs

Helpful scripts for the KodeKloud labs

*Make Nano the default editor*
`select-editor`

*Set the Kube Editor*
`export KUBE_EDITOR=nano`
`echo $KUBE_EDITOR`

*Linix Process Commands*

`-e` - display all processes
`-f` - full format - detailed listing
`ps -ef | grep kubelet`

*Vi Editor Commands*
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

*If unsure about a command*
`kubectl expain replicaset`

*Get all Kuberentes object
`kubectl get all`

*Create a pod directly*
`kubectl run podName --image=image [--dry-run=client] [-o yaml] > myfile.yaml`

*Run a Pod with arguments*
`kubectl run podName --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > myfile.yaml`

*Create from file*
`kubectl create -f myfile.yaml`

*If you want to delete and recreate*
`kubectl --force -f myfile.yaml`

*Apply changed file to an already running pod*
`kubectl apply -f myfile.yaml`

## Services
Types
* NodePort
* ClusterIP
* Load Balancer

*Change the image of a container in a pod*
`kubectl set image pod/redis redis=redis`

*Get extended information about a Pod*
`kubectl get pods -o wide --watch`

*Get pods by label*
`kubectl get pods --selector app=App1`

*Count rows*
`kubectl get pods --selector app=App1 --no-headers | wc -l`

*Create ReplicaSet file via a Deployment*
`kubectl get rs my-rs -o yaml > rs.yaml`

*ReplicaSet commands*

`kubectl create -f relicaset.yaml`

`kubectl get replicaset`

*Delete ReplicaSet*
`kubectl delete replicaset my-rs`

*Update ReplicaSet*
`kubectl replace -f replicaset.yaml`

*Scale ReplicaSet (does not change the replica in the file)*
`kubectl scale --replicas=6 -f replicaset.yaml`

*Edit ReplicaSet to scale*
`kubectl edit rs my-rs`

## Deployments

*Create a sample Deployment file
`kubectl create  deployment mydeployment --image=nginx --replicas=5 --dry-run=client -o yaml `

*Get deployments
`kubectl get deployments`

## Services

*Get help for creating a service*
`kubectl create service nodeport --help`

*Create a NodePort service*
`kubectl create service nodeport myservice --tcp=8080:80 --node-port=30000 --dry-run=client -o yaml`

## Imperative Commands

*Create an run a pod*
`kubectl run --image=nginx nginx`

*Create a deployment*
`kubectl create deployment --image=nginx nginx`

*Expose a port of the deployment*
`kubectl expose deployment nginx --port 80`

*Edit the in-memory Kubernetes file*
`kubectl edit deployment nginx`

*Scale the deployment*
`kubectl scale deployment nginx --replicas=5`

*Change the deployment image*
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

*You can add annotations to the metadata section*
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

*Taint a node*
`kubectl taint nodes mynode key=value:taint-effect`

*Check for taints*
`kubectl describe node node01 | grep taints`

## Node Selectoes

*Label a node*
`kubectl label node node01 labelKey=labelValue`

*Add nodeSelector on Pod to select bode*

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

*Define Pod resource limits*

Request - how much does the Pod need to be scheduled (minimum required)
Limit - how much is the Pod allowed to consume (upper bound)

| Request       | Limit         | Result               |
| ------------- | ------------- |  ------------- |
| No Requests   | No Limit      | Any Pod can take up all the resources of the node and other Pods can be scheduled and be starved |
| No Requests   | Limits        | Requests = Limits.   |
| Requests      | Limits        | Pods will be scheduled and limited. Pods will be limited even though resources may be available |
| Requests      | No Limits     | Each Pod will be scheduled correct and can still use resources efficiently |

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

*Define Limit Ranges*
A policy to constract the resource allocations (limits and requests) for each object (ex Pod) in a namespace

*CPU Default Limits*
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

*Memory Default Limits*
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

*Resource Quotas*
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

* Change the `kind` to DaemonSet
* Get rid of `replicas` and `strategy`

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

*Find the --config= file*
`ps -ef | grep kubelet`

* Ex: `/var/lib/kubelet/config.yaml`

Then edit the config file and look for `staticPodPath:`
`nano /var/lib/kubelet/config.yaml` 

Restart the kubelet to apply
`systemctl restart kubelet`

*View static pods*
`docker ps`

Kubectl will also list static pods
`kubectl get pods`

*How to get into a node to delete a static Pod`

*First, get the node information*
`kubectl get nodes -o wide`
Copy the internal-ip

*SSH into the node*
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

*Use scheduler in Pod*
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

*To view scheduler, look at events*
```
kubectl get events -o wide
```

*To view scheduler logs*
`kubectl logs my-customer-scheduler --namespace=kube-system

