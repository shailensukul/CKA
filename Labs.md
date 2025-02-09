# Labs

Helpful scripts for the KodeKloud labs

*Create a pod directly*
`kubectl run podName --image=image [--dry-run=client] [-o yaml] > myfile.yaml`

*Create from file*
`kubectl create -f myfile.yaml`

*Apply changed file to an already running pod*
`kubectl apply -f myfile.yaml`

*Change the image of a container in a pod*
`kubectl set image pod/redis redis=redis`

*Get extended information about a Pod*
`kubectl get pods -o wide`

*Create ReplicaSet file via a Deployment*
`kubectl create deployment my-rs --image=nginx --dry-run=client -o yaml > rs.yaml`