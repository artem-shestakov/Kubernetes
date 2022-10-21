# DaemonSets

## DS example
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: gpu-monitor
spec:
  selector:
    matchLabels:
      app: gpu-monitor
  template:
    metadata:
      labels:
        app: gpu-monitor
    spec:
      nodeSelector:
        gpu: "true"
      containers:
        - name: gpu-monitor
          image: <gpu-monitor-image>
```

## Examing DS
```shell
k get ds gpu-monitor 
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
gpu-monitor   2         2         2       2            2           gpu=true        89s

k describe ds gpu-monitor 
Name:           gpu-monitor
Selector:       app=gpu-monitor
Node-Selector:  gpu=true
Labels:         <none>
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=gpu-monitor
  Containers:
   gpu-monitor:
    Image:        <gpu-monitor-image>
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                  Message
  ----    ------            ----   ----                  -------
  Normal  SuccessfulCreate  8m41s  daemonset-controller  Created pod: gpu-monitor-qffb6
  Normal  SuccessfulCreate  8m41s  daemonset-controller  Created pod: gpu-monitor-957xs
```