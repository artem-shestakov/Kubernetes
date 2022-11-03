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

## Endpoints and Service without selector
`Endpoint` is resourse between pods and service. `Endpoints` (the resource kind is plural) defines a list of network endpoints, typically referenced by a `Service` to define which `Pods` the traffic can be sent to.
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: service-without-selector
spec:
  ports:
  - port: 53

---
apiVersion: v1
kind: Endpoints
metadata:
  name: service-without-selector # Endpoint has name same as name of service
subsets:
  - addresses:
      - ip: 8.8.8.8  # Terget IP
    ports:
      - port: 53 # Target port
```

## NodePort service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 8080
      targetPort: 80
      nodePort: 1080
```
```shell
k get svc nginx-nodeport 
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)       AGE
nginx-nodeport   NodePort   10.105.208.4   <none>        80:1080/TCP   43s
```
### Network hops
Prevent trafic hop between node and send to local pod
```yaml
...
spec:
  externalTrafficPolicy: Local
...
```

## LoadBalancer service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: load-balancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 8080
      targetPort: 80
```
```shell
k get svc load-balancer     
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
load-balancer   LoadBalancer   10.107.183.214   130.199.173.10     8080:29313/TCP   49m

k describe svc load-balancer 
Name:                     load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.107.183.214
IPs:                      10.107.183.214
Port:                     <unset>  8080/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  29313/TCP
Endpoints:                192.168.140.79:80,192.168.186.215:80,192.168.196.151:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

## ExternalName service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-name
spec:
  type: ExternalName
  externalName: google.com
  ports:
    - port: 443
```
```shell
k describe svc external-name 
Name:              external-name
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ExternalName
IP Families:       <none>
IP:                
IPs:               <none>
External Name:     google.com
Port:              <unset>  443/TCP
TargetPort:        443/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

curl -k -H "Host:google.com" https://external-name
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```
>ExternalName does not use Cluster IP. It creates CNAME to communicate with external service.

## Headless service
>Specifying `None` for the cluster IP (`.spec.clusterIP`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
```
Use DNS util to get IP addresses of pods:
```shell
nslookup nginx-headless
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginx-headless.default.svc.cluster.local
Address: 192.168.140.97
Name:   nginx-headless.default.svc.cluster.local
Address: 192.168.196.164
Name:   nginx-headless.default.svc.cluster.local
Address: 192.168.186.234
```