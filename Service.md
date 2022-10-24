# Service
## Create Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 80
    - name: https
      port: 8443
      targetPort: 443
  selector:
    app: nginx
```
>Use `sessionAffinity: ClientIP` to redirect all requests to same pod

## Examing Service
```shell
k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    10d
nginx        ClusterIP   10.99.142.139   <none>        8080/TCP   5m57s
```
