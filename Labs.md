# Labs

Helpful scripts for the KodeKloud labs

*Make Nano the default editor*
`select-editor`

*Set the Kube Editor*
`SET KUBE_EDITOR=nano`
`echo $KUBE_EDITOR`

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