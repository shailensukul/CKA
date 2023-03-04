# Kubernetes on Raspberry Pi
[Back](../README.md)

[Build Reference](https://wiki.learnlinux.tv/index.php/Setting_up_a_Raspberry_Pi_Kubernetes_Cluster_with_Ubuntu_20.04)

[Reference 2](https://medium.com/icetek/building-a-kubernetes-cluster-on-raspberry-pi-running-ubuntu-server-8fc4edb30963)

[Pi as NFS](https://gist.github.com/arkon108/86cba0b742cb8791f9fe1e9639f5205b)

# Overview

| Pi | IP |
| --- | --- |
| pi-1 | 192.168.86.36 | 
| pi-2 | 192.168.86.45 |
| pi-3 | 192.168.86.43 |
| pi-4 | 192.168.86.46 |

# Prequisites (do the following on each Raspberry Pi)

* [Download and flash Pi SD card](https://ubuntu.com/download/raspberry-pi)
* [Download Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/files/latest/download?source=navbar)

* Flash SD Card

![Flash SSD card](./images/Win32DiskImager.png)


Use ```ubuntu``` for the username and the password.

You will be asked to change this default password after you log in.

## SSH Key
Generate the key:
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/rpi -P ""
```

and add it to ssh key agent:
```
ssh-add ~/.ssh/rpi
```

Troubleshooting: 
In Windows set the "ssh-agent" Windows Service to delayed start.

Start agent with:
```
start-ssh-agent.cmd
```

Log into Pi
```
ssh ubuntu@192.168.0.238
```

Let’s log out and copy the SSH keys, so we won’t have to use the password for every login — also for k3sup to be able to deploy the Kubernetes cluster. To copy the SSH keys to the machine run the following
(Open the Git for windows command prompt)
```
ssh-copy-id -i C:/Users/shail/.ssh/rpi ubuntu@192.168.86.36
ssh-copy-id -i C:/Users/shail/.ssh/rpi ubuntu@192.168.86.45
ssh-copy-id -i C:/Users/shail/.ssh/rpi ubuntu@192.168.86.43
ssh-copy-id -i C:/Users/shail/.ssh/rpi ubuntu@192.168.86.46
```


* Edit the host name

Edit /etc/hosts and /etc/hostname on the SD card to the actual name of the instance
```
sudo nano /etc/hostname
```

Change that hostname, open and edit the necessary file and save.

Next open the hosts file with the command 
```
sudo nano /etc/hosts
```
Where you see your old hostname (i.e. localhost), change it to the new name. Save and close that file. Now, reboot your machine.

For example:
```
 k8s-master

 k8s-worker-01
```
(Or whatever naming scheme you wish)

* In Windows, edit the hosts file to point to your PI IP addresses:
Path `C:\Windows\System32\drivers\etc\hosts`

* Configure boot options

Edit /boot/firmware/cmdline.txt 
```
nano /boot/firmware/cmdline.txt
```

and add:

```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
```

Note: Add that to the end of the first line, do not create a new line.

* Install all updates

```
 sudo apt update && sudo apt dist-upgrade
 ```
 
 * Reboot

Reboot each Pi:

```
 sudo reboot
```

* Create a user for yourself
```
 sudo adduser jay
 usermod -aG sudo jay
```

* Install Docker

```
 curl -sSL get.docker.com | sh
 sudo usermod -aG docker jay
```

* Set Docker daemon options

Edit the daemon.json file (this file most likely won't exist yet)

```
 
 ```

 ```
 {
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
     "max-size": "100m"
   },
   "storage-driver": "overlay2"
 }
```

* Enable routing

Find the following line in the file: 
`nano /etc/sysctl.conf`

```
 #net.ipv4.ip_forward=1
```

Uncomment that line.

* Reboot again
```
 sudo reboot
```

* Test that docker is working properly

Check docker daemon:
```
 systemctl status docker
```

Run the hello-world container:
```
 docker run hello-world
```

# Install Kubernetes 

Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

* Add Kubernetes repository
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


* Add Kubernetes apt repository
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

```

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Master-only - Initialize Kubernetes

 Generate a bootstrap token to authenticate nodes joining the cluster
 ```
TOKEN=$(sudo kubeadm token generate)
echo $TOKEN
d584xg.xupvwv7wllcpmwjy
```

Reset a cluster, if necessary

```
kubeadm reset 
systemctl restart kubelet
pkill kubelet

rm /etc/containerd/config.toml
systemctl restart containerd
```

(Optional) Delete a cluster
```
kubectl config delete-cluster default
```

(Optional) Add firewall rules
Or run this if you do not want firewalls
`systemctl stop firewalld`

```
apt install firewalld

$ sudo firewall-cmd --permanent --add-port=6443/tcp
$ sudo firewall-cmd --permanent --add-port=2379-2380/tcp
$ sudo firewall-cmd --permanent --add-port=10250/tcp
$ sudo firewall-cmd --permanent --add-port=10251/tcp
$ sudo firewall-cmd --permanent --add-port=10252/tcp
$ sudo firewall-cmd --permanent --add-port=10255/tcp
$ sudo firewall-cmd –reload
```

Enter the following commands on each worker node:

```
$ sudo firewall-cmd --permanent --add-port=10251/tcp
$ sudo firewall-cmd --permanent --add-port=10255/tcp
$ firewall-cmd --reload
```

Init a cluster:
```
kubeadm init --token=${TOKEN} --kubernetes-version=v1.26.2 --pod-network-cidr=10.244.0.0/16  --apiserver-advertise-address=192.168.86.36
```

```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.86.45:6443 --token ${TOKEN} \
    --discovery-token-ca-cert-hash sha256:c7ecf173f263da1a74bb08e61c79a7d5e05727bd93590e4459240962497c0419
```

Once this runs, you will get some output that will include the join command, but don't join nodes yet. Copy this somewhere for later.

* Set up config directory

The previous command will give you three additional commands to run, most likely these:

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Go ahead and run those, but if it recommends different commands, run those instead.

* Install a CNI add-on - Install flannel network driver
A CNI add-on handles configuration and cleanup of the pod networks. This exercise uses the Flannel CNI add-on. With the podCIDR value already set, you can just download the Flannel YAML and use kubectl apply to install it into the cluster. This can be done on one line using kubectl apply -f - to take the data from standard input. This will create the ClusterRoles, ServiceAccounts, and DaemonSets (etc.) necessary to manage the pod networking.

```
 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
 ```

Note: The lack of sudo is intentional

* Make sure all the pods come up
```
kubectl get pods --all-namespaces
```

* Join worker nodes to the cluster

Once all of the pods have come up, run the join command on each worker node. This command was provided in an earlier step.

* Check status of nodes

See if the nodes have joined successfully, run the following command a few times until everything is ready:
```
 kubectl get nodes
```

* pod.yml
```
 apiVersion: v1
 kind: Pod
 metadata:
   name: nginx-example
   labels:
     app: nginx
 spec:
   containers:
     - name: nginx
       image: linuxserver/nginx
       ports:
         - containerPort: 80
           name: "nginx-http"
```

* service-nodeport.yml
```
 apiVersion: v1
 kind: Service
 metadata:
   name: nginx-example
 spec:
   type: NodePort
   ports:
     - name: http
       port: 80
       nodePort: 30080
       targetPort: nginx-http
   selector:
     app: nginx
```

* Apply the pod yaml file
```
 kubectl apply -f pod.yml
```
Check the status with:
```
 kubectl get pods
```
Check the status with more info:
```
 kubectl get pods -o wide
```

* Apply the service yaml file
```
 kubectl apply -f service-nodeport.yml
```

Check the status with:
```
 kubectl get service
 ```


# Upgrading Ubuntu

## Step 1 Update and upgrade your current Ubuntu 20.04 system:

```
sudo apt update 
sudo apt upgrade
sudo apt dist-upgrade
```

## Step 2 Remove all unused packages:

```
sudo apt autoremove
```

## Step 3 Install the Ubuntu upgrade utility.

In case the `do-release-upgrade` command is missing you might need to install the `update-manager-core` packages first.

Furthermore, if you are upgrading before the official upgrade from Ubuntu 18.04 is available you need to use the -d with the bellow command in order to force the direct upgrade:

```
sudo do-release-upgrade
```

## Step 4 Upgrade K3S

Manual Upgrade (run on every node)
```
# From the stable channel
curl -sfL https://get.k3s.io | sh -

# Restart servers
sudo systemctl restart k3s

# Restart agents
sudo systemctl restart k3s-agent
```

Check version
```
kubectl get nodes -o wide
```

Install the System Upgrade Controller
```
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/download/v0.6.2/system-upgrade-controller.yaml
```

Get upgrade plans 
```
kubectl -n system-upgrade get plans -o yaml
kubectl -n system-upgrade get jobs -o yaml
```

# Upgrade Ubuntu

## Update Ubuntu LTS

```
#check version 
lsb_release -a
```

```
sudo apt install update-manager-core
```

```
sudo apt update && sudo apt dist-upgrade
```

reboot 

```
#upgrade
sudo do-release-upgrade
```

## Remove Kubernetes
```
#!/bin/sh
# Kube Admin Reset
kubeadm reset

# Remove all packages related to Kubernetes
apt remove -y --allow-change-held-packages kubeadm kubectl kubelet kubernetes-cni 
apt purge -y --allow-change-held-packages kube*
sudo apt autoremove

# Remove docker containers/ images ( optional if using docker)
docker image prune -a
systemctl restart docker
apt purge -y docker-engine docker docker.io docker-ce docker-ce-cli containerd containerd.io runc --allow-change-held-packages

# Remove parts

apt autoremove -y

# Remove all folder associated to kubernetes, etcd, and docker
rm -rf ~/.kube
rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/lib/etcd2/ /var/run/kubernetes ~/.kube/* 
rm -rf /var/lib/docker /etc/docker /var/run/docker.sock
rm -f /etc/apparmor.d/docker /etc/systemd/system/etcd* 

# Delete docker group (optional)
groupdel docker

# Clear the iptables
iptables -F && iptables -X
iptables -t nat -F && iptables -t nat -X
iptables -t raw -F && iptables -t raw -X
iptables -t mangle -F && iptables -t mangle -X
```