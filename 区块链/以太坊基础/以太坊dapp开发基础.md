## 以太坊dapp开发基础

部署合约最重要的工具有如下几种：

 - Truffle：是以太坊的开发环境、测试框架和资产通道。换句话说，它可以帮助你开发、发布和测试智能合约等等。你可以阅读有关 Truffle 套件的文档，了解更多信息。

 - Ganache：以前叫作 TestRPC，它在 TestRPC 和 Truffle 的集成后被重新命名为 Ganache。Ganache 的工作很简单：**创建一个虚拟的以太坊区块链**，并生成一些我们将在开发过程中用到的虚拟账号。

 - Mist：Mist 是一个分布式网络 apps 的浏览器，相当于是只针对 Dapps 的 Chrome 或 Firefox。目前来说，它仍然是不安全的，所以你还不能在不受信任的 dapp 中使用它。

 - 以太坊钱包：它是 Mist 的一个版本，但只启动一个 dapp ——以太坊钱包。Mist 和以太坊钱包只是 UI（用户界面）前端，我们还需要一个将我们连接到以太坊区块链的核心程序（它可以是一个真正的以太坊区块链，也可以是一个测试版的）。

- Geth：Geth 是把你连接到区块链的核心应用程序，它也可以启动一个新的区块链（在我们这个示例中，我们将创建一个本地测试网区块链），创建合约，挖掘以太币等。

#### Truffle
Truffle是针对基于以太坊的Solidity语言的一套开发框架。本身基于Javascript。

Truffle对客户端做了深度集成。开发，测试，部署一行命令都可以搞定。不用再记那么多环境地址，繁重的配置更改，及记住诸多的命令。

zwlj：也就是基于solidity和以太坊多了一层封装，让你更方便的进行开发了。

### 开始
话不多说，我们从Truffle开始，试着上手dapp的开发。

我们现用npm全局安装truffle和ganache

```
npm install -g truffle
npm install -g ganache-cli
```

这两步比较快就能完成，接下来我们创建一个truffle项目：

```
truffle init
```

![](image/truffle0.png)

可以看到会产生如上目录


### 参考

[教程 | 以太坊开发演练，Part-2：Truffle，Ganache，Geth 和 Mist](https://ethfans.org/posts/ethereum-development-walkthrough-part-2)
