# Deplyments
## Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
        resources: {}
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: echo
  name: echo
spec:
  replicas: 3
  selector:
    matchLabels:
      run: echo
  template:
    metadata:
      labels:
        run: echo
    spec:
      containers:
      - name: echo
        image: jmalloc/echo-server
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
```

## Rolling update strategy
* RollingUpdate
* Recreate

## Rolling update
```shell
k set image deploy nginx nginx=httpd
```

## Rolling back
```shell
k rollout history deployment nginx   
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

k rollout undo deployment nginx --to-revision 2
deployment.apps/nginx rolled back
```