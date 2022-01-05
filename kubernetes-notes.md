# Kubernetes Notes
Mostly kubectl commands.

Run with minikube:
```
minikube start --vm-driver=virtualbox
```

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
<br>
Motivation of using services: if some set of Pods (call them "backends") provides functionality to other Pods (call them "frontends") inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?

### Exposing a Resource
We can use kubectl expose command to expose a resource as a new Kubernetes Service. That resource can be a Deployment, another Serivce, a ReplicaSet, a ReplicationController, or a Pod.

Example is to expose the ReplicaSet that's already running in the cluster:
```
kubectl expose rs go-demo-2 --name=go-demo-2-svc --target-port=28017 --type=NodePort
```
The result is the target port will be exposed on every node of the cluster to the outside world and it will be routed to one of the Pods controlled by the ReplicaSet.
<br>
Get information about the service that was imperatively created:
```
kubectl describe svc go-demo-2-svc
```
The following information was given:
```
...
Selector:                service=go-demo-2,type=backend
Type:                    NodePort
IP:                      10.0.0.194
Port:                    <unset>  28017/TCP
TargetPort:              28017/TCP
NodePort:                 <unset>  31879/TCP
Endpoints:               172.17.0.4:28017,172.17.0.5:28017
Session Affinity:        None
External Traffic Policy: Cluster
Events:                  <none>
```
Notes:
- NodePort automatically created ClusterIP type -> all the Pods in the cluster can access the TargetPort.
- The Port is set to 28017 -> that is the port that the Pods can use to access the Service. Since we did not specify the port explicitly when we executed the command, its value is the same as the value of the TargetPort, which is the port of the associated Pod that will receive all the requests.
- Serivce: provides access to Pods from inside the cluster (through Kube Proxy) or from outside the cluster (through Kube DNS).

### Simple Commands
Create a service from a file
```
kubectl create -f svc/go-demo-2-svc.yml
```
Delete a service from a file
```
kubectl delete -f svc/go-demo-2-svc.yml
```

# Deployments
Display the current-context
```
kubectl config current-context
```

We regularly add --record to the kubectl create command which allows us to track each change to our resources such as deployments. (Note: --record flag is currently deprecated)
```
kubectl create -f deploy/go-demo-2-db.yml --record
```
Why create the cluster through a Deployment if it gives the same result as creating a ReplicaSet directly? The real advantage of Deployments come from being able to upgrade/downgrade pods.
<br>
### Making Changes
Update image in pod example:
```
kubectl set image -f deploy/go-demo-2-db.yml db=mongo:3.4 --record 
```
We can also use 'kubectl edit' to update a deployment. Not the best way to edit.
```
kubectl edit -f deploy/go-demo-2-db.yml
```
We can alternatively use 'kubectl apply' command for yaml files that don't update frequently.

### Deployment Strategies
**Recreate Strategy** deployment strategy where we kill all the existing pods before an update. This strategy is much better suited for our single-replica database.

**RollingUpdate Strategy** deployment allows us to deploy new releases without downtime.

When to use what? Ask the question: would there be an adverse effect if two different versions of my application are running in parallel. An example would be to use the recreate strategy when running the database as a single replica. 

### Simple Ccmmands
1. create deployment
```
kubectl create -f deploy/go-demo-2-api.yml
```
2. Get deployment status
```
kubectl get -f deploy/go-demo-2-api.yml
```
3. Update pod image
```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:2.0
```
4. Observe rollout status
```
kubectl rollout status -w -f deploy/go-demo-2-api.yml
```
5. Describe cluster
```
kubectl describe -f deploy/go-demo-2-api.yml
```
6. See rollout history. TODO: find alternative to --record flag since it's deprecated.
```
kubectl rollout history -f deploy/go-demo-2-api.yml
```

### Rolling Back Deployments
1. Rollback to previous deployment
```
kubectl rollout undo -f deploy/go-demo-2-api.yml
```

### Deploying New Releases
Deploy new release
```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:3.0 --record
```
Revert to specific history
```
kubectl rollout undo -f deploy/go-demo-2-api.yml --to-revision=2
```

### Rolling Back Failed Deployments
Set non-existing image
```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:does-not-exist --record
```
Check ReplicaSet for type=api
```
kubectl get rs -l type=api
```
Check rollout status
```
kubectl rollout status -f deploy/go-demo-2-api.yml
```
Undo Rollout
```
kubectl rollout undo -f deploy/go-demo-2-api.yml
```

### Updating Multiple Objects
Almost everything in Kubernetes is operated using label selectors. 
Get deployment labels:
```
kubectl get deployments --show-labels
```
We want to update mongo Pods created using different-app-db and go-demo-2-db Deployments. Both are different services, but are uniquely idenitfied with the labels type=db and vendor=MongoLabs. Example of filtering deployments with same labels.
```
kubectl get deployments -l type=db,vendor=MongoLabs
```
Try updating multiple services
```
kubectl set image deployments -l type=db,vendor=MongoLabs db=mongo:3.4 --record
```

### Scaling Deployments
Change spec.replicas in deploy/go-demo-2.yml to spec.replias=5. Apply changes:
```
kubectl apply -f deploy/go-demo-2-scaled.yml
```

When scaling is frequent and, hopefully, automated, we cannot expect to update YAML definitions and push them to Git. 

The number of replicas should not be part of the design. Instead, they are a fluctuating number that changes continuously (or at least often), depending on the traffic, memory and CPU utilization, and so on.

We can use imperative commands to scale:
```
kubectl scale deployment go-demo-2-api --replicas 8 --record
```