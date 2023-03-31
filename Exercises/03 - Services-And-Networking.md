[Back](../README.md)

Lab Exercises for Services & Networking
=======================================

Lab Exercises for Services & Networking
===========================

Exercise 0 - Setup
==================

-   Prepare a cluster (Single node, kubeadm, k3s, etc)
-   Open browser tabs to <https://kubernetes.io/docs/>, <https://github.com/kubernetes/> and <https://kubernetes.io/blog/> (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))

[](https://github.com/David-VTUK/CKA-StudyGuide/blob/master/LabGuide/03-Services%20%26%20Networking.md#exercise-1---understand-connectivity-between-pods)Exercise 1 - Understand connectivity between Pods
==========================================================================================================================================================================================================

1.  Deploy [the following manifest](https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml)
2.  Using `kubectl`, identify the Pod IP addresses
3.  Determine the DNS name of the service.

Answer

[](https://github.com/David-VTUK/CKA-StudyGuide/blob/master/LabGuide/03-Services%20%26%20Networking.md#exercise-2---understand-clusterip-nodeport-loadbalancer-service-types-and-endpoint)Exercise 2 - Understand ClusterIP, NodePort, LoadBalancer service types and endpoint
==============================================================================================================================================================================================================================================================================

1.  Create three `deployments` of your choosing
2.  Expose one of these deployments with a service of type `ClusterIP`
3.  Expose one of these deployments with a service of type `Nodeport`
4.  Expose one of these deployments with a service of type `Loadbalancer`
    1.  Note, this remains in `pending` status unless your cluster has integration with a cloud provider that provisions one for you (ie AWS ELB), or you have a software implementation such as `metallb`

Answer - ImperativeAnswer - Declarative

[](https://github.com/David-VTUK/CKA-StudyGuide/blob/master/LabGuide/03-Services%20%26%20Networking.md#exercise-3---know-how-to-use-ingress-controllers-and-ingress-resources)Exercise 3 - Know how to use Ingress controllers and Ingress resources
====================================================================================================================================================================================================================================================

1.  Create an `ingress` object named `myingress` with the following specification:

-   Manages the host `myingress.mydomain`
-   Traffic to the base path `/` will be forwarded to a `service` called `main` on port 80
-   Traffic to the path `/api` will be forwarded to a `service` called `api` on port 8080

Answer - ImperativeAnswer - Declarative

[](https://github.com/David-VTUK/CKA-StudyGuide/blob/master/LabGuide/03-Services%20%26%20Networking.md#exercise-4---know-how-to-configure-and-use-coredns)Exercise 4 - Know how to configure and use CoreDNS
============================================================================================================================================================================================================

1.  Identify the configuration location of `coredns`
2.  Modify the coredns config file so DNS queries not resolved by itself are forwarded to the DNS server `8.8.8.8`
3.  Validate the changes you have made
4.  Add additional configuration so that all DNS queries for `custom.local` are forwarded to the resolver `10.5.4.223`

Answer

CKA-StudyGuide/03-Services & Networking.md at master - David-VTUK/CKA-StudyGuide - GitHub============