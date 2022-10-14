# Pod
## Attributes
* `Labels` let you group Pods and associate them with other objects in powerful ways. 
* `Annotations` let you add experimental features and integrations with 3rd-party tools and services. 
* `Probes` let you test the health and status of Pods, enabling advanced scheduling, updates, and more. 
* `Affinity` and `anti-affinity` rules give you control over where Pods run. 
* `Termination control` lets you to gracefully terminate Pods and the applications they run. 
* `Security policies` let you enforce security features. 
* `Resource requests` and `limits` let you specify minimum and maximum values for things like CPU, memory and disk IO.

## Shared linux namespaces in pod
* Network
* UTC
* IPC

## Pod manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
  namespace: default
spec:
  containers:
    - image: nginx:1.22
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
```
## Run pod
```shell
k apply -f nginx.yml 
pod/nginx created
```

## Introspecting running Pods
* `k get pods`
    * `-o` options
        * `wide` additional info
        * `yaml` pod description in YAML
```shell
# Get pods
k get pod
NAME           READY   STATUS    RESTARTS   AGE
nginx          1/1     Running   0          48s

# by labels and value
k get pod -l env=debug --show-labels 
NAME    READY   STATUS    RESTARTS   AGE     LABELS
nginx   1/1     Running   0          4h16m   app=nginx,env=debug

k get pod -l env=debug,app=nginx --show-labels
NAME    READY   STATUS    RESTARTS   AGE     LABELS
nginx   1/1     Running   0          4h51m   app=nginx,env=debug

k get pod -l env!=debug --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx-v2          1/1     Running   0          32m     app=frontend,env=prod

k get pod -l 'env in (prod,debug)' --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx             1/1     Running   0          4h49m   app=nginx,env=debug
nginx-v2          1/1     Running   0          33m     app=frontend,env=prod

k get pod -l 'env notin (prod,debug)' --show-labels

# by label name
k get pod -l env --show-labels 
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx             1/1     Running   0          4h17m   app=nginx,env=debug
nginx-v2          1/1     Running   0          2m24s   app=frontend,env=prod

# not equal label name
k get pod -l '!env' --show-labels
```
* `k describe pods`
* `k logs <pod>`
    * `--container <container>` contaier logs
```shell
k logs nginx 

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/10/13 09:46:58 [notice] 1#1: using the "epoll" event method
2022/10/13 09:46:58 [notice] 1#1: nginx/1.22.0
2022/10/13 09:46:58 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
2022/10/13 09:46:58 [notice] 1#1: OS: Linux 5.4.0-113-generic
2022/10/13 09:46:58 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/10/13 09:46:58 [notice] 1#1: start worker processes
2022/10/13 09:46:58 [notice] 1#1: start worker process 31
2022/10/13 09:46:58 [notice] 1#1: start worker process 32
```
>Container logs are automatically rotated daily and every time the log file reaches 10MB in size. The kubectl logs command only shows the log entries from the last rotation.

* `k exec <pod> -- <command>`
    * `-i` - pass stdin to container
    * `-t` - stdin is TTY

## Sending request to pod
### Services
Look `Service` file
### Port forwarding
```shell
k port-forward nginx 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

# Check by another shell
curl http://localhost:8080 -I

HTTP/1.1 200 OK
Server: nginx/1.22.0
```
## Labels
```shell
# Show labels
k get pod --show-labels 
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          26m   app=nginx

# Specify labels in output
k get pod -L app       
NAME              READY   STATUS    RESTARTS   AGE    APP  
nginx             1/1     Running   0          29m    nginx

# Add and change labels
k label pod nginx env=test

k label pod nginx env=debug --overwrite 

k get pod --show-labels 
NAME              READY   STATUS    RESTARTS   AGE     LABELS
nginx             1/1     Running   0          4h10m   app=nginx,env=debug
```

## Scheduling pod on node
### by labels
```shell
k get nodes -l gpu
NAME     STATUS   ROLES    AGE     VERSION
node01   Ready    <none>   5h17m   v1.23.1
```
```yaml
piVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
    - image: nginx:1.22
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
```
```shell
k create -f nginx-gpu.yml
k get node -o wide
NAME              READY   STATUS    RESTARTS   AGE     IP                NODE     NOMINATED NODE   READINESS GATES
...
nginx-gpu         1/1     Running   0          22s     192.168.196.130   node01   <none>           <none>
...
```
## by node name
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx-node
spec:
  nodeName: node02
  containers:
    - image: nginx:1.22
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
```
```shell
NAME              READY   STATUS    RESTARTS       AGE   IP                NODE     NOMINATED NODE   READINESS GATES
...
nginx-node        1/1     Running   0              48s   192.168.140.67    node02   <none>           <none>
```

## Pod annotation
```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx","namespace":"default"},"spec":{"containers":[{"image":"nginx:1.22","name":"nginx","ports":[{"containerPort":80,"protocol":"TCP"}]}]}}
  creationTimestamp: "2022-10-13T09:46:46Z"
  labels:
    app: nginx
    env: debug
  name: nginx
```
```shell
k annotate pod nginx mypod.local/creater=artem_shestakov

k describe pod nginx 

Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             node02/192.168.56.22
Start Time:       Thu, 13 Oct 2022 12:46:46 +0300
Labels:           app=nginx
                  env=debug
Annotations:      mypod.local/creater: artem_shestakov
```

## Delete pods
```shell
# Delete one pod
kubectl delete pod nginx

# Delete several pods
kubectl delete pod nginx1 nginx2

# Delete by labels
k delete pod -l env=debug

# Delete by delete namespace
k delete ns nginx-namespace

#Delete all in namespace
kubectl delete pods --all

# Delete all in all
k delete all --all
```

## Pod probes
### Liveness probe
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  containers:
    - image: nginx:1.22
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
      livenessProbe:
        initialDelaySeconds: 5
        periodSeconds: 30
        failureThreshold: 3
        httpGet:
          path: /
          port: 80
```