## Kubectl Cheatsheet

| Syntax | Description |
| ----------- | ----------- |
| Pods |
| Get Pods | `kubetl get pods ` |
| Get Pods (details with Nodes) | `kubetl get pods -o w --all-namespaces` |

## Install kubeadm

```
# Update the apt package index and install packages needed to use the Kubernetes apt repository:
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Download the Google Cloud public signing key:
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the Kubernetes apt repository:
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


## Upgrading Kubernetes

* Upgrade primary control plane node

```
# Determine which version to upgrade to
apt update
apt-cache madison kubeadm
# find the latest 1.22 version in the list
# it should look like 1.22.x-00, where x is the latest patch

# get current version
kubeadm version

# replace x in 1.22.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.22.x-00 && \
apt-mark hold kubeadm
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.22.x-00
```

* Upgrade additional control plane nodes
* Upgrade worker nodes

