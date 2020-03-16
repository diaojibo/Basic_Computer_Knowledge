## k8s使用基础

Kubernetes 跟 Docker 等很多项目最大的不同，就在于它不推荐你使用命令行的方式直接运行容器（虽然 Kubernetes 项目也支持这种方式，比如：kubectl run），而是希望你用 YAML 文件的方式，即：把容器的定义、参数、配置，统统记录在一个 YAML 文件中，然后用这样一句指令把它运行起来：



```
$ kubectl create -f 我的配置文件
```



这么做最直接的好处是，你会有一个文件能记录下 Kubernetes 到底“run”了什么。比如下面这个例子：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

```

像这样的一个 YAML 文件，对应到 Kubernetes 中，就是一个 API Object（API 对象）。当你为这个对象的各个字段填好值并提交给 Kubernetes 之后，Kubernetes 就会负责创建出这些对象所定义的容器或者其他类型的 API 资源。

可以看到，这个 YAML 文件中的 Kind 字段，指定了这个 API 对象的类型（Type），是一个 Deployment。所谓 Deployment，是一个定义多副本应用（即多个副本 Pod）的对象。Deployment 还负责在 Pod 定义发生变化时，对每个副本进行滚动更新（Rolling Update）。在上面这个 YAML 文件中，我给它定义的 Pod 副本个数 (spec.replicas) 是：2。

定义了一个 Pod 模版（spec.template），这个模版描述了我想要创建的 Pod 的细节。在上面的例子里，这个 Pod 里只有一个容器，这个容器的镜像（spec.containers.image）是 nginx:1.7.9，这个容器监听端口（containerPort）是 80。**Pod 就是 Kubernetes 世界里的“应用”；而一个应用，可以由多个容器组成。**

每一个 API 对象都有一个叫作 Metadata 的字段，这个字段就是 API 对象的“标识”，即元数据，它也是我们从 Kubernetes 里找到这个对象的主要依据。这其中最主要使用到的字段是 Labels。

比如，在上面这个 YAML 文件中，Deployment 会把所有正在运行的、携带“app: nginx”标签的 Pod 识别为被管理的对象，并确保这些 Pod 的总数严格等于两个。