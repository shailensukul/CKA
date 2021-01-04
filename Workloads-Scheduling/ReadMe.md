# 15% - Workloads & Scheduling
[Back](../README.md)

* [Understand deployments and how to perform rolling update and rollbacks](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Use [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) and [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) to configure applications
    -   [configure a pod with a configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
    -   [configure a pod with a secret](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)
* Know how to [scale applications]
(https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
    -   [scaling a statefulset](https://kubernetes.io/docs/tasks/run-application/scale-stateful-set/)
    -   [scaling a replicaset](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#scaling-a-replicaset)
* Understand the primitives used to create robust, self-healing, application deployments
    -   [Replicaset](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
    -   [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
    -   [Statefulsets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
    -   [Daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
* Understand [how resource limits can affect Pod scheduling](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#how-pods-with-resource-requests-are-scheduled)
* Awareness of manifest management and common templating tools
    -   [Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
        -   [Kustomize Blog](https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/)
    -   [manage kubernetes objects](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/)
    -   [Install service catalog using helm](https://kubernetes.io/docs/tasks/service-catalog/install-service-catalog-using-helm/)
        -   Non-k8s.io resource: CNCF Kubecon video: [An introduction to Helm - Bridget Kromhout, Microsoft & Marc Khouzam, City of Montreal](https://youtu.be/x2w6T0sE50w?list=PLj6h78yzYM2O1wlsM-Ma-RYhfT5LKq0XC)
    -   Non-k8s.io resource: External resource: [templating-yaml-with-code](https://learnk8s.io/templating-yaml-with-code)