# 10% - Storage
[Back](../README.md)

* Understand [storage classes](https://kubernetes.io/docs/concepts/storage/storage-classes/), [persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* Understand [volume mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-mode), [access modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) and [reclaim policies](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy) for volumes
* Understand [persistent volume claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) primitive
* Know how to [configure applications with persistent storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

<details>
<summary>
StorageClass, PersistentVolume, and PersitentVolumeClaim examples
</summary>
```
#### Storage Class example
#
#### Persistent Volume Claim example
#

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage-sc
  resources:
    requests:
      storage: 100Mi

## Persistent Volume example
#
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 200Mi
  local:
    path: /data/pv/disk021
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage-sc
  volumeMode: Filesystem

###  Pod using the pvc
#
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
      - name: local-persistent-storage
        mountPath: /var/www/html
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```
</details>