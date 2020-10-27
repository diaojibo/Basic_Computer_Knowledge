## 命令行工具

kubectl，是k8s的集群命令行工具，通过kubectl能够对集群本身进行管理，并能在集群上进行容器化应用的安装部署。



![Kubectlå¸¸ç¨å½ä»¤è¯¦è§£](https://user-gold-cdn.xitu.io/2019/12/1/16ec1496f0d808d6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 查看集群状态

```
kubectl version --short=true 查看客户端及服务端程序版本信息
kubectl cluster-info 查看集群信息
```

#### 查看资源对象

```
kubectl get namespace 查看命名空间
kubectl get pods,services -o wide (-o 输出格式 wide表示plain-text)
kubectl get pod -l "key=value,key=value" -n kube-system (-l 标签选择器(多个的话是与逻辑)，-n 指定命名空间，不指定默认default)
kubectl get pod -l "key1 in (val1,val2),!key2" -L key (-l 基于集合的标签选择器, -L查询结果显示标签) 注意：为了避免和shell解释器解析!,必须要为此类表达式使用单引号
kubectl get pod -w(-w 监视资源变动信息)
```

#### 创建资源对象

```
kubectl run name --image=(镜像名) --replicas=(备份数) --port=(容器要暴露的端口) --labels=(设定自定义标签)
kubectl create -f **.yaml  陈述式对象配置管理方式
kubectl apply -f **.yaml  声明式对象配置管理方式（也适用于更新等）
```

