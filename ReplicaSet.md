# ReplicaSet

## RS example
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
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

## Examing RS
```shell
k get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   3         3         3       4m4s

k describe rs
Name:         nginx
Namespace:    default
Selector:     app=nginx
Labels:       <none>
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.22
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  23m   replicaset-controller  Created pod: nginx-pckrh
  Normal  SuccessfulCreate  23m   replicaset-controller  Created pod: nginx-dzkmm
  Normal  SuccessfulCreate  23m   replicaset-controller  Created pod: nginx-vfjvm
```

## Use matchExpressions
```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
        - nginx
```

## Delete RS
```shell
k delete rs nginx 
replicaset.apps "nginx" deleted
```