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
`kubectl get pods -o wide`

*Create ReplicaSet file via a Deployment*
`kubectl get rs my-rs -o yaml > rs.yaml`

*ReplicaSet commands*

`kubectl create -f relicaset.yaml`

`kubectl get replicaset`

*Delete ReplicaSet*
`kubectl delete replicaset my-rs`

*Update ReplicaSet*
`kubectl replace -f replicaset.yaml`

*Sacle ReplicaSet (does not change the replica in the file)*
`kubectl scale --replicas=6 -f replicaset.yaml`

*Edit ReplicaSet to scale*
`kubectl edit rs my-rs`

## Deployments

*Create a sample Deployment file
`kubectl create  deployment mydeployment --image=nginx --replicas=5 --dry-run=client -o yaml `

*Get deployments
`kubectl get deployments`