# Deployments
[Back](./Services.md)

A [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

## Use Case
The following are typical use cases for Deployments:

-   [Create a Deployment to rollout a ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment). The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
-   [Declare the new state of the Pods](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
-   [Rollback to an earlier Deployment revision](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment) if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
-   [Scale up the Deployment to facilitate more load](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment).
-   [Pause the rollout of a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment) to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
-   [Use the status of the Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#deployment-status) as an indicator that a rollout has stuck.
-   [Clean up older ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#clean-up-policy) that you don't need anymore

## Creating a Deployment

The following is an example of a Deployment. It creates a ReplicaSet to bring up three nginx Pods:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## Commands

Create a deployment
`kubectl apply -f nginx-deployment.yaml`

Check status of deployments
`kubectl get deployments`

Check deployment rollout status
`kubectl rollout status deployment/nginx-deployment`

## Updating a Deployment

Update the deployment image
`kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1`

Or edit the deployment template and update `.spec.template.spec.containers[0].image`

`kubectl edit deployment/nginx-deployment`


Describe Deployments
`kubectl describe deployments`

Pause Deployment
`kubectl rollout pause deployment/nginx-deployment`

Then update the image of the Deployment:
`kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1`

Notice that no new rollout started
`kubectl rollout history deployment/nginx-deployment`

Resume deployment
`kubectl rollout resume deployment/nginx-deployment`



