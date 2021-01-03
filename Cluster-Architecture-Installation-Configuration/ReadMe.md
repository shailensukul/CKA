# 25% - Cluster Architecture, Installation & Configuration
[Back](../README.md)

## Manage role based access control (RBAC)


## Use Kubeadm to install a basic cluster
Installing Kubernetes v1.19 with Kubeadm
----------------------------------------

I'm not sure how detailed the cluster creation process will be on the CKA exam will be but I will be starting from the point that Docker is installed and `kubeadm` and `kubectl` need to be downloaded and installed. We'll do that with the following steps:

### Install Dependencies

```
sudo apt update && sudo apt install -y apt-transport-https curl
```

### Get Google's GPG Key for Kubernetes Repository

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg  | sudo apt-key add -
```

### Add Kubernetes Source to apt

```
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

### Update apt and install kubelet, kubeadm, and kubectl

```
sudo apt update && sudo apt install -y kubelet kubeadm kubectl
(optional) sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize Kubernetes Cluster on Control Plane Node

Then we'll simply run` sudo kubeadm init`. This simple command is very powerful and loaded! In short, it downloads, installs, and configures kube-scheduler, kube-proxy, kube-controller-manager, kube-apiserver, etcd, and coredns, generates certificates and keys. Full details on kubeadm init process can be found in the [Kubernetes documentation](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/).

Upon completion, you'll see:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.117:6443 --token 29wfr4.u88qkj7at954cnij\
    --discovery-token-ca-cert-hash sha256:2aefd220fd9e1a1ad7e15cc68c1f0d3331a07858cf676c8e4f15611b87801b26
```

The output illustrates 3 important steps that are necessary post-configuration:

#### Add kubeconfig

These steps assume the user has never administered a Kubernetes cluster before and thus needs a `$HOME/.kube` directory for kubeconfig. It also copies `/etc/kubernetes/admin.conf` that was created as a template for the kubeconfig to that local directory and changes the ownership to match the local user. Without this step, it won't be possible to communicate with kube-apiserver. I wrote a little more about kubeconfig in a [previous post](https://brandonwillmott.com/2020/10/01/important-directories-to-know-for-kubernetes-cka-exam#kubeconfig).

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Set Up Pod Networking

There's a [myriad of container network interfaces (CNIs)](https://kubernetes.io/docs/concepts/cluster-administration/addons/) that can be configured in a Kubernetes cluster. For the exam, you should know at least one if the question asks you to configure a CNI in the cluster. Most recommend weave for the CKA since you can find (and bookmark!) the URL for the weave YAML in [Kubernetes documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node). Let's use that:

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Output:

```
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

Inspecting the pods in kube-system, we can see that weave-net pods are running:

![](https://bdwill.files.wordpress.com/2020/10/kubectl-get-po-kube-system-with-weave.png?w=497)