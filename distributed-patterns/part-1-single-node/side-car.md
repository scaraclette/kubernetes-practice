# Side Car Pattern
The sidecar pattern is a single-node pattern made up of two containers. Which are the application container and sidecar container.

### Example
Adding a sidecar container to readout resource usage. Add /topz as sidecar container: https://hub.docker.com/r/brendanburns/topz

1. Run/deploy any docker container:
```
docker run -d -p 80:80 --name=web nginx
```
2. Get app id for web
3. Run topz example:
```
docker run --pid=container:${APP_ID} -p 8080:8080 brendanburns/topz:db0fa58 /server --addr=0.0.0.0:8080
```