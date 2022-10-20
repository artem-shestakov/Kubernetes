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