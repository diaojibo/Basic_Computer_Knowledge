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

### 智能合约
Hyperledger Fabric智能合约被称为 **chaincode**，当一个区块链外部的一个应用程序需要访问账本时，就会调用chaincode。大多数情况下，chaincode只会访问账本的数据库组件和世界状态(world state)（比如查询），但不会查询交易记录。

### 隐私
根据网络的需求，在一个Business-to-Business（B2B）网络中的参与者会对信息共享的程度极为敏感。然而，对于其他的网络，隐私并不是首要考虑的因素。

Hyperledger Fabric支持构建隐私保护严格的网络，也支持构建相对开放的网络。




### 参考
[Hyperledger中文文档](https://hyperledgercn.github.io/hyperledgerDocs/blockchain_zh/)
