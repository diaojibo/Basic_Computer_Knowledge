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

![](image/fabric3.png)

### run the network
我们进到fabric sample的first network文件夹。

这个文件夹下已经有一个启动脚本`byfn.sh`，可以快速部署4个peer节点分属两个不同的org，并且还会启动一个容器服务，用以部署chaincode和加入peer到channel。

跟着文档走的话，很快就能启动一个网络。只需要执行

```
./byfn.sh -m generate
```

然后不断地按yes，即可。但是我们还是要慢慢缕清楚整个fabric网络启动的过程。

从脚本启动后输出的日志来看，启动网络主要有这么几个过程：

1. cryptogen加密生成
2. 创建引导排序服务
3. 创建channel通道，设置交易配置
4. 启动Org1的节点
5. 启动Org2的节点


接下来我们会在步骤上分析fabric网络的部署

## fabric网络部署

### 加密生成器crptogen
在MSP基础笔记中初步了解MSP概念之后，我们知道节点都需要证书来证明自己的身份。

我们将使用cryptogen工具为我们生成各种网络实体的加密材料（x509证书）。这些证书是身份的代表，它们允许在我们的网络实体进行交流和交易时进行签名/验证身份验证。

zwlj：也就是说，我们将使用这个工具，为各个节点创建一个证书。

要使用这个证书，我们首先需要写一个**crypto-config.yaml**配置文件，大致内容如下：

![](image/fabric11.png)

除了上面配置之外，还有count变量：

![](image/fabric12.png)

接下来我们要在了解一下交易生成器这个工具

### 配置交易生成器
这个二进制工具**configtxgen**也是帮助我们生成一些初始配置用的，比如创世区块的配置，channel的配置，peer的配置。

这个生成器对应的配置文件是**configtx.yaml**

### 手动生成配置文件
在之前的案例里，我们运行了byfn.sh来集成我们这些操作。

#### 运行cryptogen
之前有介绍这个工具是用来产生证书的，假如我们在配置文件的目录下，手动执行工具，则有：

![](image/fabric13.png)

命令要打sudo，可以看到命令行输出了两个名字：

```
org1.example.com
org2.example.com
```

接着在项目文件夹下，我们就可以看到生成的配置文件夹crypto-config,里面会有分别的order配置文件夹和org对应的文件夹。

## 参考
[官方实例文档](https://hyperledger-fabric.readthedocs.io/en/release-1.3/install.html)
