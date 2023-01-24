## Kubectl Cheatsheet

| Syntax | Description |
| ----------- | ----------- |
| Pods |
| Get Pods | `kubetl get pods ` |
| Get Pods (details with Nodes) | `kubetl get pods -o w --all-namespaces` |

## Install kubeadm

```
# Update the apt package index and install packages needed to use the Kubernetes apt repository:
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Download the Google Cloud public signing key:
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the Kubernetes apt repository:
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


## Upgrading Kubernetes

* Upgrade primary control plane node

```
# Determine which version to upgrade to
apt update
apt-cache madison kubeadm
# find the latest 1.22 version in the list
# it should look like 1.22.x-00, where x is the latest patch

# get current version
kubeadm version

# replace x in 1.22.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.22.x-00 && \
apt-mark hold kubeadm
-
# since apt-get version 1.1 you can also use the following method
apt-get update && \
apt-get install -y --allow-change-held-packages kubeadm=1.22.x-00
```

* Upgrade additional control plane nodes
* Upgrade worker nodes

| Command     | Description | 
| ----------- | ----------- |
| HELP | |
| kubectl get pods --show-labels | Show labels for pods |
| kubectl get pod kubia-123 -o yaml | Display the pod definition in YAML |
| kubectl explain pod  |  Print documentation    |
| CREATE | |
| kubectl create -f kubia-manual.yaml | Create an object based on the YAML definition |
| LABEL | |
| kubectl label pod kubia-manual creation_method=manual| Update the lable for a pod |
| kubectl label pod kubia-manual env=debug --overwrite | Change existing label |
| kubectl get pod -l creation_method=manual | filter pods by label |
| kubectl get pod -l creation_method=manual,label2=blah | filter pods by labels |
| kubectl get pod -l creation_method!=manual | filter pods which do not have the label value |
| kubectl get pod -l env | filter pods which have the the label key |
| kubectl get pod -l '!env' | filter pods which do not have the the label key |
| kubectl label node node1 gpu=true | Label a node. The the pods YAML can have spec: nodeSelector: gpu: "true" to only deploy to this node | 
| ANNOTATIONS | |
| kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"| Make an annotation|
| kubectl describe pod kubia-manual | View annotations |
| KUBERNETES NAMESPACES | Kubernetes scope for object names |
| kubectl get ns | Get namespaces|
| kubectl get pods --namespace kube-system | List pods in a namespace |
| kubernetes create -f custom-namespace.yaml | Create a custom namespace from : apiVersion: v1 kind: Namespace metadata: name: -namespace |
| kubectl create namespace custom-namespace | Create namespace |
| kubectl create -f kubia-manual.yaml -n custom-namespace | Create a pod in a namespace or add namespace: custom-namespace in the metadata section of YAML|
| DIAGNOSTICS | |
| `kubectl run test --image=luksa/kubectl-proxy -n foo` | Create and run a particular image, in a namespace. Creates a deployment or job to manage the created container(s). |
| kubectl logs kubia-manual | Get logs of a running process |
| kubectl port-forward kubia-manual 8888:8080 | Forward a local port to a specific node for debugging |
| kubectl logs kubia --previous| Get application logs of the previous crashed container |
| kubectl describe pod kubia | Gets the reason for why a container was restarted |
| DELETE | |
| kubectl delete pod custom-pod | Delete a pod | 
| kubectl delete pod -l creation_method=manual | Delete pods using label selectors |
| kubectl delete ns custom-namespace | Delete a namespace along with all its pods |
| kubectl delete pod -all | Delete all pods but keep the namespace |
| kubectl delete all -all | Delete all pods and the ReplicationController |
| REPLICATION CONTROLLER | *Are deprecated* |
| kubectl get rc | Get Replication Controller |
| kubectl describe rc kubia-rc | Describe the Replication Controller |
| kubectl edit rc kubia-rc | Open the ReplicationController's YAML file in the text editor |
| kubectl scale rc kubia-rc --replicas=3 | Scale the ReplicationController |
| kubectl delete rc kubia-rc | Delete the ReplicationController and all its Pods |
| kubectl delete rc kubia-rc --cascade=false | Delete the ReplicationController but leave all its Pods |
| REPLICASETS | *More expressive than a ReplicationController* |
| kubectl get rc kubia-rs | Get ReplicaSet |
| kubectl describe rc kubia-rs | Describe the ReplicaSet in more detail |
| kubectl delete rs kubia | Delete the ReplicaSet |
| DAEMONSET | *One pod per node* |
| kubectl get ds | Gets all DeamonSets |
| kubectl label node myNode disk=ssd | Label the node so that it can picked by the DaemonSet selector |
| JOBS | |
| kubectl get jobs | Gets running job |
| kubectl get pods --show-all | Shows the completed pod for the job. They are not deleted after they complete |
| kubectl logs batch-job-28gfq | Get the logs for the pod that the job created |
| kubectl describe job batch-job | Check on the status of the job |
| kubectl delelte job batch-job | Delete a job |
| kubectl scale job batch-job --replicas=3 | Runs 3 pods in parallel |
| DEPLOYMENTS | |
| Create a deployment | `kubectl apply -f nginx-deployment.yaml` |
| Check status of deployments | `kubectl get deployments` |
| Check deployment rollout status | `kubectl rollout status deployment/nginx-deployment` |
| Update the deployment image | `kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1` |
| Describe Deployments | `kubectl describe deployments` |
| Pause Deployment | `kubectl rollout pause deployment/nginx-deployment` |
| Resume Deployment | `kubectl rollout resume deployment/nginx-deployment` |
| Show Deployment history | `kubectl rollout history deployment/nginx-deployment`|
| SERVICES | | 
| Create service | ` kubectl create -f kubia-svc.yaml` | 
| Get services | `kubectl get service` | 
| Describe the service | `kubectl describe svc kubia` | 
| Delete service | `kubectl delete service kubia` | 
| ENDPOINTS | | 
| Get service endpoints | `kubectl get endpoints <servicename>` | 
| VOLUMES | | 
| Get persistence volumes  | `kubectl get pv` |
| Get persistent volume claims | `kubectl get pvc` |
| List storage classes | `kubectl get sc`  |
| DIAGNOSTICS| |
|  Attach to a process that is already running inside an existing container | `kubectl attach POD -c CONTAINER` | 
|  Execute a command in a container (creates an additional procewss) | `kubectl exec POD [-c CONTAINER] -- COMMAND [args...]` | 
| Forward one or more local ports to a pod | `kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N]`<br>ex `# Listen on port 8888 locally,forwarding to 5000 in the pod` <br>`kubectl port-forward mypod 8888:5000` | 
| List components and their statuses | kubectl get componentstatuses | 
| SERVICE ACCOUNTS| | 
| Get service accounts | `kubectl get sa` | 
| SECRETS | | 
| Get secrets | `kubectl get secrets` | 
| NAMESPACE | |
| Get namespace | `kubectl get ns` |
| Create namespace | `kubectl create ns foo` |
| DISABLE RBAC| |
| Disable RBAC | `kubectl delete clusterrolebinding permissive-binding` |
| ROLE| |
| Create a role | `kubectl create role service-reader --verb=get --verb==list --resources=services -n bar` |
| Get roles | `kubectl get roles` |
| ROLE BINDING | |
| Create role binding | `kubectl create rolebinding test --role=service-reader --serviceaccount=foo:default -n foo` |
| To bind a Role to a user instead of ServiceAccount, use the --user argument. To bind it to a group, use --group| |
| Get role binding | `kubectl get rolebinding test -n foo -o yaml` |
| Role binding references a SINGLE role | |
| Role binding can reference multiple subjects| |
| CLUSTER ROLE | |
| Create a cluster role | `kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes` |
| Get cluster role | `kubectl get clusterrole pv-reader -o yaml` |
| Get cluster roles | `kubectl get clusterroles` |
| CLUSTER ROLE BINDING | |
| Create cluster role binding | `kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=foo:default` |
| Get clusterrolebings | `kubectl get clusterrolebindings` |
| | |

