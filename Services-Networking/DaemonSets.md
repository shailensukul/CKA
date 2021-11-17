# DaemonSets
DaemonSets are used for running exactly 1 pod on each and every node in the cluster.

A DaemonSet deploys pods to all nodes in the cluster, unless you specify that the pods should only run on a subset of all the nodes.
This is done bhy specifying the `node-Selector` property in the pod template.

Note: DaemonSets bypass the scheduler completely.



