# CKA
Certified Kubernetes Administrator V 1.19

* [CKA Exam Site](https://www.cncf.io/certification/cka/)
* [Curriculum](https://github.com/cncf/curriculum)
* [Kubernetes Documentation](https://kubernetes.io/docs)
* [Other Guide](http://www.lib4dev.in/info/walidshaari/Kubernetes-Certified-Administrator/103684801)

## Installation
[Install on Pi Cluster](kubernetes-on-pi.md)

## [25% - Cluster Architecture, Installation & Configuration](./Cluster-Architecture-Installation-Configuration/ReadMe.md)

* Manage role based access control (RBAC)
* Use Kubeadm to install a basic cluster
* Manage a highly-available Kubernetes cluster
* Provision underlying infrastructure to deploy a Kubernetes cluster
* Perform a version upgrade on a Kubernetes cluster using Kubeadm
* Implement etcd backup and restore

## [15% - Workloads & Scheduling](./Workloads-Scheduling)
* Understand deployments and how to perform rolling update and rollbacks
* Use ConfigMaps and Secrets to configure applications
* Know how to scale applications
* Understand the primitives used to create robust, self-healing, application deployments
* Understand how resource limits can affect Pod scheduling
* Awareness of manifest management and common templating tools

## [20% - Services & Networking](./Services-Networking/ReadMe.md)
* Understand host networking configuration on the cluster nodes
* Understand connectivity between Pods
* Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
* Know how to use Ingress controllers and Ingress resources
* Know how to configure and use CoreDNS
* Choose an appropriate container network interface plugin

## [10% - Storage](./Storage/ReadMe.md)
* Understand storage classes, persistent volumes
* Understand volume mode, access modes and reclaim policies for volumes
* Understand persistent volume claims primitive
* Know how to configure applications with persistent storage

## [30% - Troubleshooting](./Troubleshooting/ReadMe.md)
* Evaluate cluster and node logging
* Understand how to monitor applications
* Manage container stdout & stderr logs
* Troubleshoot application failure
* Troubleshoot cluster component failure
* Troubleshoot networking


Tips:
-----

practice pratice pratice

Get familiar with:

-   Familiarize yourself with the documentation, initially [concepts](https://kubernetes.io/docs/concepts/) and mostly [tasks](https://kubernetes.io/docs/tasks/), kubectl explain command, [kubectl cheatsheet](https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/), and [kubectl commands reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

-   https://kubernetes.io/docs/concepts/

-   https://kubernetes.io/docs/tasks/

-   https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/

-   https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

-   `kubectl api-versions` and `kubectl api-resources` wih `grep` for a specific resoruce e.g. pv, pvc, deployment, storageclass, ..etc can help figure out the apiVersion, and kind combined with explain below will help in constructing the yaml manifest

-   [kubectl explain --recurisve](https://blog.heptio.com/kubectl-explain-heptioprotip-ee883992a243) to construct out any yaml manifest you need and find its specd and details

-   When using kubectl for investigations and troubleshooting utilize the wide output it gives your more details

```
     $kubectl get pods -o wide  --show-labels  --all-namespaces
     or
     $kubectl get pods -o wide  --show-labels  -A     # -A is quicker than --all-namespaces

```

-   In `kubectl` utilizie `--all-namespaces or better -A` to ensure deployments, pods, objects are on the right name space, and right desired state

-   for events and troubleshooting utilize kubectl describe if its pod/resource related and logs if it is application issue related

```
     $kubectl describe pods <PODID>   # for pod, deployment, other k8s resource issues/events
     $kubectl logs <PODID>            # for container/application issues like crash loops

```

-   [fast with kubectl](https://medium.com/faun/be-fast-with-kubectl-1-18-ckad-cka-31be00acc443) e.g. the '-o yaml' in conjuction with `--dry-run=client` allows you to create a manifest template from an imperative spec, combined with `--edit` it allows you to modify the object before creation

```
kubectl create service clusterip my-svc -o yaml --dry-run=client > /tmp/srv.yaml
kubectl create --edit -f /tmp/srv.yaml

```

-   use kubectl [aliases](https://github.com/ahmetb/kubectl-aliases) to speed up and reduce typo errors, practice these alaises early at your work and study for the exam. some example aliases:

```
alias k='kubectl'
alias kg='kubectl get'
alias kgpo='kubectl get pod'
alias kcpyd='kubectl create pod -o yaml --dry-run=client'
alias ksysgpo='kubectl --namespace=kube-system get pod'

alias kd='kubectl delete'
alias kdf='kubectl delete -f'
## for quick deletes you can add --force --grace-period=0  **Not sure if it is a good idea if you are in a production cluster**
alias krmgf='kubectl delete --grace-period 0 --force'
alias kgsvcoyaml='kubectl get service -o=yaml'
alias kgsvcwn='watch kubectl get service --namespace'
alias kgsvcslwn='watch kubectl get service --show-labels --namespace'

#example usage of aliases
krmgf nginx-8jk71    # kill pod nginx-8jk71 using grace period 0 and force

```

Miscellaneous (resources not allowed during exam):
--------------------------------------------------

1.  [Troubleshooting use cases by Ian/Container solutions](https://github.com/ContainerSolutions/kubernetes-examples)

Popular training and practice sites:
------------------------------------

*Double check if the course is uptodate with the latest exam information (e.g. api, or curicuilim)*

-   [Mumshad CKA with practice tests and mock exams](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/) - Highly recommended
-   [Killer.sh CKA simulator](https://killer.sh/cka) ⟹ use code walidshaari for 20% discount - they update frequently
-   [LinuxAcademy/ACloudGuru CKA course](https://acloud.guru/learn/7f5137aa-2d26-4b19-8d8c-025b22667e76) # labs last checked were updated to 1.18
-   [rx-m online CKA course](https://rx-m.com/cka-online-training/)
-   [Pluralsight CKA course](https://www.pluralsight.com/paths/certified-kubernetes-administrator)
-   Duffie Cooly [hands-on CKA video](https://k8s.work/cka-lab.mp4) using KinD and accompanying [notes](https://hackmd.io/@mauilion/cka-lab)
-   [Stilian Stoilov](https://www.linkedin.com/in/stilian-stoilov-379972a9/) [practice questions](https://github.com/StenlyTU/K8s-training-official) - 40+ tasks with increasing difficulty.
