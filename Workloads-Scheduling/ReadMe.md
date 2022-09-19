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

## ConfigMaps and Secrets

ConfigMap is a map containing key/value pairs with values ranging from literals to full config files.
The contents of a ConfigMap are passed to containers as either environment variables or as files in a volume.

```
apiVersion: v1
data:
    sleep-interval: "25"
kind: ConfigMap
metadata:
    name: fortune-config
    namespace: default
```

Create the ConfigMap
`kubectl create -f fortune-config.yaml`

Creating ConfigMap from a file.
kubectl will create an individual map entry for each file in the specified directory, but only for files whose name is a valid ConfigMap key.

`kubectl create configmap my-config --from-file=/path/to/dir`

```
kubectl create configmap myconfig
--from-file=foo.json // a single file
--from-file=bar=fobar.conf // a file stored under a custom key
--from-file=config-opts/ // a whole directory
--from-literal=some=thing // a literal value
```


Pass ConfigMap to Pod

```
apiVersion: v1
kind: Pod
metadata:
    name: fortune-env-from-configmap
spec:
    containers:
    - image: luksa/fortune:env
    env:
    - name: INTERVAL
    valueFrom:
        configMapKeyRef:
            name: fortune-config
            key: sleep-interval
            optional: false // if optional is true, the container will start even if the ConfigMap does not exist
```

Mapp all values from a ConfigMap to the Pod

```
...
spec:
    containers:
    - image: some-image
    envFrom: // Use envForm instead of emv
    - prefix: CONFIG_ // Optional. All environment variables will be prefixed with CONFIG_
      configMapRef:
        name: my-config-map // Name of ConfigMap
...
```

Referencing environment variables as arguments

```
apiVersion: v1
kind: Pod
metadata:
    name: fortune-env-from-configmap
spec:
    containers:
    - image: luksa/fortune:env
    env:
    - name: INTERVAL
    valueFrom:
        configMapKeyRef:
            name: fortune-config
            key: sleep-interval
            optional: false // if optional is true, the container will start even if the ConfigMap does not exist

    args: ["$(INTERVAL)"]
```

