# Replication Controllerå

## Parts of Replica Controller
* Label selector
* Pod count
* Pod template

## Manifest example
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.22
          ports:
            - containerPort: 80
              protocol: TCP
```

## Change RC
```shell
k edit rc nginx
replicationcontroller/nginx edited

k get pod --show-labels 
NAME          READY   STATUS    RESTARTS      AGE   LABELS
nginx-5pqtj   1/1     Running   1 (31m ago)   17h   app=nginx
nginx-bkvjb   1/1     Running   1 (31m ago)   17h   app=nginx
nginx-cnszr   1/1     Running   1 (32m ago)   21h   app=nginx,type=special

k delete pod nginx-5pqtj 
pod "nginx-5pqtj" deleted

k get pod --show-labels 
NAME          READY   STATUS    RESTARTS      AGE   LABELS
nginx-bkvjb   1/1     Running   1 (31m ago)   17h   app=nginx
nginx-cnszr   1/1     Running   1 (32m ago)   21h   app=nginx,type=special
nginx-sh5s4   1/1     Running   0             5s    app=nginx,stage=prod
```
>After `edit` RC you need delete pod to update it.

## Scaling RC
```shell
k scale rc nginx --replicas 10

k get rc nginx
NAME    DESIRED   CURRENT   READY   AGE
nginx   10        10        10      21h
```

## Delete RC
```shell
# with pods
k delete rc nginx

# without pods
k delete rc nginx --cascade=orphan
```