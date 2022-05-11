# 20% - Services & Networking
[Back](../README.md)

## My Notes

* [Replication Controllers And Other Controllers](./ReplicationControllersAndOtherControllers.md)

* [Services](./Services.md)

* [ReplicaSets](./ReplicaSets.md)

* [DeamonSets](./DaemonSets.md)

## Sections

* Understand [host networking configuration on the cluster nodes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
* Understand [connectivity between Pods](https://kubernetes.io/docs/concepts/workloads/pods/#pod-networking)
* Understand ClusterIP, NodePort, LoadBalancer [service](https://kubernetes.io/docs/concepts/services-networking/service/) types and endpoints
* Know how to use [Ingress controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) and [Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)
    - [Ingress concepts](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* Know how to [configure and use CoreDNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)
* Choose an appropriate [container network interface plugin](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)
    -   [Kubernetes Networking Intro and Deep-Dive - Bowei Du & Tim Hockin, Google](https://youtu.be/tq9ng_Nz9j8)
    -   [Kubernetes and Networks: why is this so dang hard?](https://youtu.be/xB190-yyJnY?t=241)
    -   [Kubecon Eu 2020 Tutorial: Communication Is Key - Understanding Kubernetes Networking - Jeff Poole, Vivint Smart Home](https://youtu.be/InZVNuKY5GY?list=PLj6h78yzYM2O1wlsM-Ma-RYhfT5LKq0XC)

## Services Notes
[Chapter 5 - Services: enabling clients to discover and talk to pods](./Services.md)



## Cheatsheet
| Command       | Example           | 
| ------------- |:-------------:| 
| Create a resource from file | `kubectl create -f <fname>` | 
| Update a resource from file | `kubectl apply -f <fname>` | 