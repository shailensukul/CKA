[Back](../README.md)


[Nano Editor](./Nano-Editor-Tricks.md)

# Tips

## Reference material

* https://kubernetes.io/docs
* https://github.com/kubernetes
* https://kubernetes.io/blog

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

ex: `kubectl explain deployment`

# Exercises

* [01 Cluster Architcture, Installation and Configuration](01-Cluster Architcture, Installation and Configuration.md)