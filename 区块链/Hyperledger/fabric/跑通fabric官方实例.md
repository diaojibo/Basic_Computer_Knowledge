# 跑通fabric官方实例
以fabric 1.3为例，我们尝试去跑通官方给出的fabric 例子。

## 执行拉取脚本
执行官方给出的这个命令


```
curl -sSL http://bit.ly/2ysbOFE | sudo bash -s 1.3.0
```

![](image/fabric2.png)

执行这个命令可厉害了，会先去拉取官方的fabric examples项目，然后接着会拉取部署fabric所需要的docker镜像。如上图，其实就是：

```
===> List out hyperledger docker images
hyperledger/fabric-ca          1.4.0-rc1           ebef9b071211        2 days ago          244MB
hyperledger/fabric-ca          latest              ebef9b071211        2 days ago          244MB
hyperledger/fabric-zookeeper   0.4.14              d36da0db87a4        2 months ago        1.43GB
hyperledger/fabric-zookeeper   latest              d36da0db87a4        2 months ago        1.43GB
hyperledger/fabric-kafka       0.4.14              a3b095201c66        2 months ago        1.44GB
hyperledger/fabric-kafka       latest              a3b095201c66        2 months ago        1.44GB
hyperledger/fabric-couchdb     0.4.14              f14f97292b4c        2 months ago        1.5GB
hyperledger/fabric-couchdb     latest              f14f97292b4c        2 months ago        1.5GB
hyperledger/fabric-javaenv     1.3.0               2476cefaf833        2 months ago        1.7GB
hyperledger/fabric-javaenv     latest              2476cefaf833        2 months ago        1.7GB
hyperledger/fabric-tools       1.3.0               c056cd9890e7        2 months ago        1.5GB
hyperledger/fabric-tools       latest              c056cd9890e7        2 months ago        1.5GB
hyperledger/fabric-ccenv       1.3.0               953124d80237        2 months ago        1.38GB
hyperledger/fabric-ccenv       latest              953124d80237        2 months ago        1.38GB
hyperledger/fabric-orderer     1.3.0               f430f581b46b        2 months ago        145MB
hyperledger/fabric-orderer     latest              f430f581b46b        2 months ago        145MB
hyperledger/fabric-peer        1.3.0               f3ea63abddaa        2 months ago        151MB
hyperledger/fabric-peer        latest              f3ea63abddaa        2 months ago        151MB
```

上面下载下来的镜像列表，我们用 `docker image ls`也能看到，一行代表一个instance，我们会发现每个image id都会有两个instance实例。

执行完命令后我们进入example目录，bin文件夹下，我们可以看到命令帮我们下载了很多需要用到的二进制程序，这些二进制程序其实都是platform-specific的。

## 参考
[官方实例文档](https://hyperledger-fabric.readthedocs.io/en/release-1.3/install.html)
