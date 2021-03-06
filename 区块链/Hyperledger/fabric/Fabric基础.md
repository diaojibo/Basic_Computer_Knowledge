## Fabric
2015年，Linux基金会启动了Hyperledger项目，目标是发展跨行业的区块链技术。Hyperledger项目并不仅仅是定义一个单一的区块链标准，它更鼓励通过开源社区的力量协作开发区块链技术。

Hyperledger Fabric是Hyperledger中的一个区块链项目。与其他区块链技术类似，Hyperledger Fabric包含一个账本，**使用智能合约** 并且是一个通过所有参与者管理交易的系统。

**Hyperledger Fabric与其他区块链系统最大的不同体现在私有和许可。** 与开放无需许可的网络系统允许未知身份的参与者加入网络不同（需要通过工作量证明协议来保证交易有效并维护网络的安全），**Hyperledger Fabric通过Membership Service Provider(MSP)来登记所有的成员。**

Hyperledger Fabric也提供了多个可拔插选项。账本数据可被存储为多种格式，共识机制可被接入或者断开，同时支持多种不同的MSP。

Hyperledger Fabric提供了建立`channel`的功能，这允许参与者为交易新建一个单独的账本。当网络中的一些参与者是竞争对手时，这个功能变得尤为重要。因为这些参与者并不希望所有的交易信息——比如提供给部分客户的特定价格信息——都对网络中所有参与者公开。只有在同一个channel中的参与者，才会拥有该channel中的账本，而其他不在此channel中的参与者则看不到这个账本。

### 共享账本
Hyperledger Fabric包含一个账本子系统，这个子系统包含两个组件：世界状态(world state)和交易记录。在Hyperledger Fabric网络中的每一个参与者都拥有一个账本的副本。

世界状态组件描述了账本在特定时间点的状态，它是账本的数据库。交易记录组件记录了产生世界状态当前值的所有交易，它是世界状态的更新历史。那么，账本则是世界状态数据库和交易历史记录的集合。

账本的世界状态存储数据库是可更换的。默认配置下，这是一个key-value存储数据库。交易记录模块不需要被接入。只需要记录在区块链网络中账本数据库被使用时之前和之后的值就可以了。

zwlj：World State是一组变量的集合，包含着transactions的执行结果。

### 智能合约
Hyperledger Fabric智能合约被称为 **chaincode**，当一个区块链外部的一个应用程序需要访问账本时，就会调用chaincode。大多数情况下，chaincode只会访问账本的数据库组件和世界状态(world state)（比如查询），但不会查询交易记录。

### 隐私
根据网络的需求，在一个Business-to-Business（B2B）网络中的参与者会对信息共享的程度极为敏感。然而，对于其他的网络，隐私并不是首要考虑的因素。

Hyperledger Fabric支持构建隐私保护严格的网络，也支持构建相对开放的网络。

### 另外一些术语

 - **资产**，资产是任何具有价值的事物。资产具有状态和所有权。在 Hyperledger Fabric 中，资产以键/值对集合的形式来表示(可以理解为世界状态WorldState的一部分)。

 - **通道**，由节点集合组成的一种逻辑结构。该功能支持一组节点创建独立的交易账本。(zwlj:也就是一组节点逻辑上构成一个独立的账本的话，那就是一个通道)

 - **组织**，Hyperledger Fabric 网络是根据属于网络成员的不同组织所拥有并贡献的节点而构建的。正是由于组织将各自的资源贡献给集体网络，网络才会存在。节点具有的身份（数字证书），是由其所属组织的成员服务提供者所分配的。不同组织的节点可位于相同通道上。

 - **成员服务提供者 (MSP)**。MSP 是作为认证中心实施的，负责管理用于对成员身份和角色进行认证的证书。在 Hyperledger Fabric 网络中，身份未知就无法参与交易。它负责管理用户 ID，并对网络上的所有参与方进行认证，使 Hyperledger Fabric 能够作为私有许可网络来运行。

 - **排序服务**。排序服务将交易打包到区块中，以便交付至通道上的节点。它负责保证网络中交易的正常交付。它可与节点和背书节点进行通信。支持的排序服务配置机制包括 Solo 和 Kafka。

 - **endorse**：背书。通常我们引申为对某个事情负责。在我们的共识机制的投票环节里，背书意味着参与投票。

 - **endorsement policy**：背书策略。由智能合约chaincode选择哪些peer节点参与到背书环节来。

 - **peer**：存放区块链数据的结点，同时还有endorse和commit功能。

### 架构

#### 应用解决方案

![](image/fabric1.png)

在区块链解决方案中，Hyperledger Fabric 网络充当后端，通过应用前端与网络进行通信。SDK 可帮助设置前端与后端之间的通信，例如，Nodejs SDK 和 Java SDK。SDK 为执行用户链码、在网络中履行交易、监控事件等提供了途径。

要编写区块链应用，就需要：

1. 使用支持的编程语言（如 Go）编写链码。
2. 在 Hyperledger Fabric 网络上部署链码。
3. 使用 SDK 开发客户端应用。

#### Peer和Orderer

其实Fabric中，组件只有两个：Peer和Orderer，它们就是两个二进制程序。每个Peer中都存放全量的数据（账本），也就是完整的链。**Orderer则是用来形成共识的，也就是跟矿工差不多**。

执行“查询（query）”操作，也就是调用合约中“没有写操作”的接口的时候，只需要指定Peer的地址。调用会执行“写操作”的接口时，还需要另外指定Orderer的地址。

#### System chain

Fabric中有一组特殊的链，叫做“**system chain**”。这组链相当于整个Fabric网络的配置文件，里面记录了所有的Channel信息、参与者的信息。（每个Channel对应一条system chain）。部署Fabric时使用的创世块，就是system chain中的第一个区块。

每个参与者可以将自己的一个Peer地址写入到system chain中，这样的Peer被称为“锚点”（Anchor Peer）。Orderer们从system chain中获得Anchor Peer(实际上应该是Leader Peer)的地址，并将形成的共识通知给它们。

之后Anchor Peer通过Gossip协议，将结论八卦给其它的Peer。

Orderer中的链与Peer中的链是不同的。Oderer的链中存放的是Channel的配置，是system chain，Peer的链中存放的是Channel的数据。比对一下它们的内容就可以知道。

Peer与Orderer交织，数据更新请求在Orderer之间来回穿梭，最终被送往了Anchor Peer，大嘴巴的Anchor Peer转身通知了身边的所有Peer。

组成网络的每一个Peer和Orderer，都是得到了批准的、实名的，想要访问这个网络的用户也需要得到批准，并实名签署自己的操作。

### 交易流程
去中心化的设计，必然需要通过投票（多数大于少数）来维持数据一致性，而任何投票都必须经历以下三个过程：

1. 有一方先提出议案proposal，该议案有对应的一批投票者需要对该结果背书，这些投票者依据各自的习惯投票，并将结果反馈；
2. 统计投票结果，若获得多数同意，才能进行下一步；
3. 将获得多数同意的议案记录下来，且公之于众。



### 参考
[Hyperledger中文文档](https://hyperledgercn.github.io/hyperledgerDocs/blockchain_zh/)

[超级账本HyperLedger：Fabric掰开揉碎，一文解惑](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2018/06/25/hyperledger-fabric-main-point.html)