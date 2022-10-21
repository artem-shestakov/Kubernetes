# Job

## Job example
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
        - name: batch-job
          image: ubuntu
```

## Examing Job
```shell
k get job
NAME        COMPLETIONS   DURATION   AGE
batch-job   0/1           12s        12s

k get pod
NAME                READY   STATUS      RESTARTS   AGE
batch-job-f5nmh     0/1     Completed   0          35s

k get job         
NAME        COMPLETIONS   DURATION   AGE
batch-job   1/1           13s        37s

k logs pods/batch-job-f5nmh
```

## Job rules
### Run Job multiple times
```yaml
spec:
  completions: 5
```

### Run Job's pod in parallel
```yaml
spec:
  completions: 5
  parallelism: 2 
```

### Scaling Job
```shell
k scale job batch-job --replicas 10
```

### Limit Job time
```yaml
spec:
  activeDeadlineSeconds: 10
```