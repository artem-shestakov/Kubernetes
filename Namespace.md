# Namespace
## Get namespaces
```shell
k get namespaces 

NAME              STATUS   AGE
default           Active   3d18h
kube-node-lease   Active   3d18h
kube-public       Active   3d18h
kube-system       Active   3d18h
```

## Create namespace
### by manifest
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-namespace
```
```shell
k create -f namespace-nginx.yml
```
### by kubectl
```shell
k create namespace test
namespace/test created
```

## Create object in namespace
>Use `--namespace` or `-n` flag
```shell
k create -f nginx.yml -n nginx-namespace 
pod/nginx created

k get pod -n nginx-namespace 
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          16s
```

>Change context alias: `alias kcd='kubectl config set-context $(kubectl config current-context) --namespace '`