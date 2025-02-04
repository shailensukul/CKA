# Labs

Helpful scripts for the KodeKloud labs

*Create a pod directly*
`kubectl run podName --image=image [--dry-run=client] [-o yaml]`

*Change the image of a container in a pod*
`kubectl set image pod/redis redis=redis`

*Get extended information about a Pod*
`kubectl get pods -o wide`