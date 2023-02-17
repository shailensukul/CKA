[Back](ReadMe.md)

# Scenarios

* Create a single control plane cluster with version 1.19
* Add a worker node to the cluster
* Upgrade the cluster to 1.20 version
* Make the cluster HA by adding additional control plane. We will also add an extra worker node.
* Upgrade the cluster to 1.21
* Etcd backup and restore

## Create a single control plane cluster with version 1.19

```

-sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```