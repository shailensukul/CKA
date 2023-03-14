[Back](../README.md)

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
* Find the `etcd-master` pod running etcdctl
`kubectl get pod -n kube-system`
* Get the pods properties
`kubectl describe pod etcd-pi-1 -n kube-system`
* Now that you have all the parameters, you can plug it into the documentation templates

Check etcd cluster health

Find the command by running 

`ETCDCTL_API=3 etcdctl  -h` to find the command for endpoint status

 `ETCDCTL_API=3 etcdctl endpoint status -h` to get the command for the endpointy health

```
ETCDCTL_API=3 etcdctl endpoint health --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key --cluster=true 
```

List members
```
ETCDCTL_API=3 etcdctl  member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key
```

Health of cluster

* Go to the [Kubernetes docs home page](https://kubernetes.io/docs/home/)
* Search for `api health` and click on the first result

```
curl -k https://localhost:6443/livez?verbose
```

## Exercise 4 - Perform a version upgrade on a Kubernetes cluster using Kubeadm

1. Using kubeadm, upgrade a cluster to the lastest version

If held, unhold the kubeadm version
```
sudo apt-mark unhold kubeadm
```

Upgrade the kubeadm version:
```
sudo apt-get install --only-upgrade kubeadm
```

plan the upgrade:
```
sudo kubeadm upgrade plan
```

```
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     1 x v1.19.0   v1.20.2
```

Upgrade the cluster
```
kubeadm upgrade apply v1.20.2
````

Upgrade Kubelet:
```
sudo apt-get install --only-upgrade kubelet
```

## Exercise 5 - Implement etcd backup and restore

1. Take a backup of etcd
2. Verify the etcd backup has been successful
3. Restore the backup back to the cluster

To work out the etcdctl arguments

* Find the `etcd-master` pod running etcdctl
`kubectl get pod -n kube-system`
* Get the pods properties
`kubectl describe pod etcd-pi-1 -n kube-system`

Use the --peer-* values
```
 Command:
      etcd
      --advertise-client-urls=https://192.168.86.36:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --experimental-watch-progress-notify-interval=5s
      --initial-advertise-peer-urls=https://192.168.86.36:2380
      --initial-cluster=pi-1=https://192.168.86.36:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.86.36:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.86.36:2380
      --name=pi-1
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
* Now that you have all the parameters, you can plug it into the documentation templates

Commands
* Go to the [Kubernetes docs home page](https://kubernetes.io/docs/home/)
* Search for `etcd backup` and click on the first result
* Search the page for `backup` and copy the command template:

Take a snapshot
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> snapshot save <backup-file-location>
```

* Swap parameters
```
ETCDCTL_API=3 etcdctl snapshot save etcd-backup-01.bak --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/peer.crt --key=/etc/kubernetes/pki/etcd/peer.key
```

Verify backup
```
ETCDCTL_API=3 etcdctl --write-out=table snapshot status etcd-backup-01.bak
```

Perform a restore:
```
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 snapshot restore etcd-backup-01.bak
```