## Job和CronJob

在 2015 年 Borg 论文刚刚发布的时候，Kubernetes 项目并不支持对 Batch Job 的管理。直到 v1.4 版本之后，社区才逐步设计出了一个用来描述离线业务的 API 对象，它的名字就是：Job。

Job API 对象的定义非常简单，我来举个例子，如下所示：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: resouer/ubuntu-bc 
        command: ["sh", "-c", "echo 'scale=10000; 4*a(1)' | bc -l "]
      restartPolicy: Never
  backoffLimit: 4
```

在这个 Pod 模板中，我定义了一个 Ubuntu 镜像的容器（准确地说，是一个安装了 bc 命令的 Ubuntu 镜像），它运行的程序是：

`echo "scale=10000; 4*a(1)" | bc -l`

这其实就是一个计算π值的容器。而通过 scale=10000，我指定了输出的小数点后的位数是 10000。这个计算大概用时 1 分 54 秒。

但是，跟其他控制器不同的是，Job 对象并不要求你定义一个 spec.selector 来描述要控制哪些 Pod。

这个 Job 对象在创建后，它的 Pod 模板，被自动加上了一个 controller-uid=< 一个随机字符串 > 这样的 Label。而这个 Job 对象本身，则被自动加上了这个 Label 对应的 Selector，从而 保证了 Job 与它所管理的 Pod 之间的匹配关系。

```
$ kubectl describe jobs/pi
Name:             pi
Namespace:        default
Selector:         controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
Labels:           controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
                  job-name=pi
Annotations:      <none>
Parallelism:      1
Completions:      1
..
Pods Statuses:    0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
                job-name=pi
  Containers:
   ...
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  1m           1m          1        {job-controller }                Normal      SuccessfulCreate  Created pod: pi-rq5rl

```

而 Job Controller 之所以要使用这种携带了 UID 的 Label，就是为了避免不同 Job 对象所管理的 Pod 发生重合。需要注意的是，**这种自动生成的 Label 对用户来说并不友好，所以不太适合推广到 Deployment 等长作业编排对象上。**

几分钟后计算结束，这个 Pod 就会进入 Completed 状态：

```
$ kubectl get pods
NAME                                READY     STATUS      RESTARTS   AGE
pi-rq5rl                            0/1       Completed   0          4m

```

这也是我们需要在 Pod 模板中定义 restartPolicy=Never 的原因：离线计算的 Pod 永远都不应该被重启，否则它们会再重新计算一遍。

在 Job 的 API 对象里，有一个 spec.activeDeadlineSeconds 字段可以设置最长运行时间