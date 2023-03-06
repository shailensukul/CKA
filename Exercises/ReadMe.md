[Back](../README.md)


[Nano Editor](./Nano-Editor-Tricks.md)

# Tips

## Use --all-namespaces to list objects from all namespaces

## Enable autocompletion

`apt-get install bash-completion`

`kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null`

## Generate template file
You can generate a template as per below:
`kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml`

`nano pod.yaml -miclET2 `

## Manual

`kubectl explain <resourceType>`

ex: `kubecto explain deployment`

# Exercises

## Exercise 1 -RBAC

1. Create a namespace called `rbac`
2. Create a service account called `job-inspector` for the `rbac` namespace
3. Create a role that has rules to `get` and `list` the job objects
4. Create a rolebinding that binds the service account `job-inspector` to the role created in step 3
5. Prove that the `job-inspector` service account can "get" `job` objects but not `deployment` objects

Answer
```
kubectl create namespace rbac
kubectl create serviceaccount job-inspector -n rbac
kubectl create role job-inspector --verb=get --verb=list --resource=jobs -n rbac
kubectl create rolebinding permit-job-inspector --role=jobs-inspector --serviceaccount=rbac:job-inspector -n rbac
kubectl --as=system:serviceaccont:rbac:job-inspector auth can-i get job -n rbac
kubectl --as=system:serviceaccount:rbac:job-inspector auth can-i get deployment -n rbac
```

