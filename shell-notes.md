## Shell Notes

### Eval
eval is a built-in linux command which is used to execute arguments as a shell command.
It combines arguments into a single string and uses it as an input to the shell and execute the commands.

### minikube ssh
ssh into minikube

### eval $(minikube docker-env)
Evaluated minikube variables so that our local Docker client is using Docker server running inside the VM.