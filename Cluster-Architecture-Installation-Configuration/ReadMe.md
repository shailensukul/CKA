# 25% - Cluster Architecture, Installation & Configuration
[Back](../README.md)

* [Manage role based access control (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

* [Use Kubeadm to install a basic cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

* [Manage a highly-available Kubernetes cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

* [Provision underlying infrastructure to deploy a Kubernetes cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

* [Perform a version upgrade on a Kubernetes cluster using Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

* [Implement etcd backup and restore](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)

[Kubecon Europe 2020: Kubeadm deep dive](https://www.youtube.com/watch?v=DhsFfNSIrQ4&feature=youtu.be)


# Installation

To initialize the control-plane node run:

```
kubeadm init <args>
```

sample commands used during backup/restore/update of nodes

```
#etcd backup and restore brief
export ETCDCTL_API=3  # needed to specify etcd api versions, not sure if it is needed anylonger with k8s 1.19+
etcdctl snapshot save -h   #find save options
etcdctl snapshot restore -h  #find restore options

## possible example of save, options will change depending on cluster context, as TLS is used need to give ca,crt, and key paths
etcdctl snapshot save /backup/snapshot.db  --cert=/etc/kubernetes/pki/etcd/server.crt  --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt

# evicting pods/nodes and bringing back node back to cluster
kubectl drain  <node># to drain a node
kubectl uncordon  <node> # to return a node after updates back to the cluster from unscheduled state to Ready
kubectl cordon  <node>   # to not schedule new pods on a node

#backup/restore the cluster (e.g. the state of the cluster in etcd)

# upgrade kubernetes worker node
kubectl drain <node>
apt-get upgrade -y kubeadm=<k8s-version-to-upgrade>
apt-get upgrade -y kubelet=<k8s-version-to-upgrade>
kubeadm upgrade node config --kubelte-version <k8s-version-to-upgrade>
systemctl restart kubelet
kubectl uncordon <node>

#kubeadm upgrade steps
kubeadm upgrade plan
kubeadm upgrade apply
```