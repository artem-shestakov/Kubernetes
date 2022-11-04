# Ingress
## Ingress-nginx on bare metal
* Download controller manifest
```shell
curl -O https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/baremetal/deploy.yaml
```
*  Update `ingress-nginx-controller` service by adding `nodePort` to access to controller by node port. Example:
```yaml
...
  - appProtocol: http
    name: http
    port: 80
    nodePort: 10080
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    nodePort: 10443
    protocol: TCP
    targetPort: https
...
```
or modify `Deployment` to `DaemonSet` to accept all connections on host port. DaemonSet good choice for service option `externalTrafficPolicy: Local`.   
>You can add `hostPort` to accept connection by host port

* Create `Ingress` object. Example:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginx
spec:
  ingressClassName: nginx
  rules:
  - host: my.nginx.local
    http:
      paths:
      - path: /
        pathType: Exact
        backend:
          service:
            name: nginx
            port: 
              number: 8080
```
## TLS
* Generate certificate and key
```shell
openssl genrsa -out tls.key 2048
$ openssl req -new -x509 -key tls.key -out tls.cert -days 360 -subj /CN=my.echo.local
```
* Create Secret with certificate and key
```shell
kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key
```
* Add `spec.tls` Ingress section
```yaml
tls:
  - hosts:
      - my.echo.local
    secretName: tls-secret
```