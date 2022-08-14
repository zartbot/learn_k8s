


```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```bash

[root@k8s-master01 ~]# kubectl create -f job.yaml
cronjob.batch/hello created
[root@k8s-master01 ~]#
[root@k8s-master01 ~]#
[root@k8s-master01 ~]# kubectl get cj
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          6s
...
[root@k8s-master01 ~]# kubectl get cj
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     1        6s              62s

[root@k8s-master01 ~]# kubectl get po
NAME                      READY   STATUS              RESTARTS   AGE
hello-27667386--1-84j4m   0/1     ContainerCreating   0          10s
[root@k8s-master01 ~]# kubectl get jobs
NAME             COMPLETIONS   DURATION   AGE
hello-27667386   1/1           15s        22s

[root@k8s-master01 ~]# kubectl logs -f hello-27667386--1-84j4m
Tue Aug  9 11:06:14 UTC 2022
Hello from the Kubernetes cluster

[root@k8s-master01 ~]# kubectl delete cronjob hello
cronjob.batch "hello" deleted
```