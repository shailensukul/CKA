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

ex: `kubecto explain deployment`

# Exercises

## Exercise 1 - RBAC

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

## Exercise 2 - Use Kubeadm to install a basic cluster
1. On a node, install kubeadm and stand up the control plane, using 10.244.0.0/16 as the pod network CIDR, and https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml as the CNI

2. On a node, install kubeadm and join it to the cluster as a worker node

3. On the master node, determine the health of the cluster by probing the API endpoint

Answer

```
# Assuming kubeadm, kubectl and kubelet have been installed
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

...
# join node as worker, get this from the previous step
kubeadm join 172.16.10.210:6443 --token 9tjntl.10plpxqy85g8a0ui \
    --discovery-token-ca-cert-hash sha256:381165c9a9f19a123bd0fee36fe36d15e918062dcc94711ff5b286ee1f86b92b 

# validate this by running
kubectl get nodes
```

## Exercise 3 - Manage a highly-available Kubernetes cluster

1. Using `etcdctl`, determine the health of the cluster
2. Using `etcdctl`, identify the list of members
3. On the master node, determine the health of the cluster by probing the API endpoint
4. Backup the etcd cluster

Answer

#### Etcd/Etcdctl

To work out the etcdctl arguments
* Go to the [Kubernetes docs home page](https://kubernetes.io/docs/home/)
* Find the `etcd-master` pod running etcdctl
`kubect get pod -n kube-system`
* Get the pods properties
`kubectl describe pod etcd-pi-1 -n kube-system`
* Now that you have all the parameters, you can plug it into the documentation templates

#### Backup
* Search for `etcd backup` and click on the first result
* Search the page for `backup` and copy the command template:
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save <backup-file-location>
```
* Swap parameters
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key snapshot save etcd-backup-01.bak
```

Verify backup
```
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --write-out=table snapshot status etcd-backup-01.bak
```

Check etcd cluster health
```
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --cluster=true endpoint health

List members
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --cacert=/etc/kubernetes/pki/etcd/ca.crt member list

Health of cluster
curl -k https://localhost:6443/healthz?verbose
```

