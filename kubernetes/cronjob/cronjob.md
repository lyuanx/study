# 名词解释：CronJob

## CronJob

CronJob即定时任务，就类似于Linux系统的crontab，在指定的时间周期运行指定的任务。在Kubernetes 1.5，使用CronJob需要开启batch/v2alpha1 API，即–runtime-config=batch/v2alpha1。

## CronJob Spec

- .spec.schedule指定任务运行周期，格式同Cron
- .spec.jobTemplate指定需要运行的任务，格式同Job
- .spec.startingDeadlineSeconds指定任务开始的截止期限
- .spec.concurrencyPolicy指定任务的并发策略，支持Allow、Forbid和Replace三个选项

```code
apiVersion: batch/v2alpha1
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
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```code
$ kubectl create -f cronjob.yaml
cronjob "hello" created
```

当然，也可以用kubectl run来创建一个CronJob：

```code
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```

```code
$ kubectl get cronjob
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         <none>
$ kubectl get jobs
NAME               DESIRED   SUCCESSFUL   AGE
hello-1202039034   1         1            49s
$ pods=$(kubectl get pods --selector=job-name=hello-1202039034 --output=jsonpath={.items..metadata.name} -a)
$ kubectl logs $pods
Mon Aug 29 21:34:09 UTC 2016
Hello from the Kubernetes cluster

# 注意，删除cronjob的时候不会自动删除job，这些job可以用kubectl delete job来删除
$ kubectl delete cronjob hello
cronjob "hello" deleted
```

## 参考

[k8s官网中文](https://www.kubernetes.org.cn/cronjob)
