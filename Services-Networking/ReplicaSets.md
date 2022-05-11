# ReplicaSets

[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) are a new generation of ReplicationController and replaces it completeley.

A ReplicaSet behaves exactly like a ReplicationController, *but it has more expressive pod selectors*.

A ReplicaSet is linked to its Pods via the Pods' **metadata.ownerReferences** field

Label selector operators:
* In - Label's value must match one of the specified values
* NotIn - Label's value must not match any of the specified values
* Exists - Pod must include a label with the specified key. The value is not important and you shouldn't specify the `values` fields
* DoesNotExist - Pod must not include a label with the specified key. You shouldn't specify the `values` fields

Note: If you specify multiple expressions or labels, all labels must match and all expressions must evaluate to true for the Pod to match the selector.

Create ReplicaSet
```
kubectl create -f kubia-replicaset.yaml
```

Show Replicasets
```
kubectl get rs
```

```
kubectl describe rs
```

Cleanup
```
kubectl delete rs kubia
```

