# Kubernetes Notes
Mostly kubectl commands.

# General Architecture
- When you deploy Kubernetes, you get a cluster: https://kubernetes.io/docs/concepts/overview/components/
A cluster is divided into Control Plane and Nodes
### Control Plane Components
The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfield). Contains the following:
- kube-apiserver
- etcd
- kube-scheduler
- kube-controller-manager
- cloud-controller-manager
### Node Components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.
- kubelet
- kube-proxy
- container runtime

# Pods
### Labels
Labels are key/value paris that are attached to objects, such as pods.
Labels are intended to be used to specify identifying attributes of objects that are meaningfule and relevant to users, but do not directly imply semantics to the core system.

### Commands
- kubectl describe pod <POD_NAME>: sub-command returned details of the specified resource.
- kubectl get pods -o wide: more detailed information.
- -f means filename

### Executing a New Process
Create short lived process (deprecated command)
```
kubectl exec <pod_name> ps aux
```
Enter into a container shell (deprecated command)
```
kubectl exec -it db sh
```
Updated commands: https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/
Note that the short options -i and -t are the same as the long options --stdin and --tty. 

Updated command to enter into a container shell
```
kubeclt exec -it <POD_NAME> -- sh
```
Opening a shell when a Pod has more than one container
```
kubectl exec -it <POD_NAME> -- container <CONTAINER_NAME> -- sh
```

Get the names of the containers from a yaml file. Will it work if the pod doesn't exist? If the pod is deleted, it will give not found error from Kubernetes server.
```
kubectl get -f pod/go-demo-2.yml -o jsonpath="{.spec.containers[*].name}"
```

### Liveness Probe
LivenessProbe can be used to confirm whether a container should be running. If the probe fails, Kubernetes will kill the container and apply restart policy which defaults to Always.


# ReplicaSets

### Selector
We use selectors to select which pods should be included in the ReplicaSet. It does not distringuish between the pods created by a ReplicaSet or some other process.

### MatchLabels
matchLabels is a map of {key, value} pairs. 

### Sample Commands
After creating a replica set, we can do the following commands:
1. List all the ReplicaSets in the cluster
```
kubectl get rs
```
2. List all the replicas only from a specified file
```
kubectl get -f rs/go-demo-2.yml
```
3. List the pods newly created by the replica set
```
kubectl get pods
```
4. Get information from specific file
```
kubectl describe -f rs/go-demo-2.yml
```
5. List out pod labels
```
kubectl get pods --show-labels
```

### ReplicaSet self healing properties
ReplicaSets and Pods are loosely coupled objects with matching labels, we can remove one without deleting the other.
1. Remove the replica set without removing the created pods (deprecated command for --cascade=false)
```
kubectl delete -f rs/go-demo-2.yml --cascade=orphan
```

Try deleting a pod
```
kubectl delete $POD_NAME
```
### Re-using the same pods
Adding --save-config argument allows us to save the configuration of the ReplicaSet thus allowing us to perform a few additional operations later on.
<br>
What happens when we don't add --save-config? We won't be able to do the 'apply' command. The apply command automatically saves the configuration so that we can edit it later on. The warning:
```
Warning: resource replicasets/go-demo-2 is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
replicaset.apps/go-demo-2 configured
```

# Services
... We need a stable, never-to-be-changed address that will forward requests to whichever Pod is currently running. Kubernetes Services provide addresses through which associated Pods can be accessed.

### Exposing a Resource
We can use kubectl expose command to expose a resource as a new Kubernetes Service. That resource can be a Deployment, another Serivce, a ReplicaSet, a ReplicationController, or a Pod.

Example is to expose the ReplicaSet that's already running in the cluster:
```
kubectl expose rs go-demo-2 --name=go-demo-2-svc --target-port=28017 --type=NodePort
```
The result is the target port will be exposed on every node of the cluster to the outside world and it will be routed to one of the Pods controlled by the ReplicaSet.