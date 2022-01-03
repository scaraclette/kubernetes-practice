## Shell Notes

### Eval
eval is a built-in linux command which is used to execute arguments as a shell command.
It combines arguments into a single string and uses it as an input to the shell and execute the commands.

### minikube ssh
ssh into minikube

### eval $(minikube docker-env)
Evaluated minikube variables so that our local Docker client is using Docker server running inside the VM.

### ps aux
The ps aux command is a tool to monitor processes running on your Linux System.

### Get a pod name
```
POD_NAME=$(kubectl get pods -o name | tail -1)
```

### Check if service is running
Get PORT from existing service
```
PORT=$(kubectl get svc go-demo-2-svc -o jsonpath="{.spec.ports[0].nodePort}")
```
Get minikube IP
```
IP=$(minikube ip)
```
Open tab
```
open "http://$IP:$PORT"
```